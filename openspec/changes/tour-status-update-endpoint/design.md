## Context

The tour admin dashboard's status update feature is broken. The current `tourService.updateTourStatus()` sends a `multipart/form-data` body with only `id` and `status` to `PUT /api/tour` — the same endpoint used for full tour updates. That endpoint's `[FromForm]` parameters include `string tourName`, `string shortDescription`, and `string longDescription` as non-nullable fields, causing ASP.NET model binding to produce empty strings. FluentValidation's `UpdateTourCommandValidator` then fails because `NotEmpty()` rejects the empty values, resulting in a 500 `ValidationException`.

The fix is a dedicated endpoint: `PUT /api/tour/{id}/status` accepting JSON `{ "status": <int> }`. This follows the **Single Responsibility Principle** — status update is a distinct use case from full tour update.

## Goals / Non-Goals

**Goals:**
- Create `PUT /api/tour/{id}/status` as a minimal, focused endpoint for status-only updates
- Fix the 500 `ValidationException` when updating tour status
- Use `ExecuteUpdateAsync` for maximum performance (no entity load, single SQL UPDATE)
- Maintain audit trail (`LastModifiedBy`, `LastModifiedOnUtc`)
- Preserve existing full-tour update behavior unchanged

**Non-Goals:**
- Changing the full-tour update endpoint behavior
- Modifying any other tour field besides `Status`
- Database schema changes
- Adding new domain events (outbox already handles this)

## Decisions

### 1. New endpoint `PUT /api/tour/{id}/status` vs modifying existing endpoint

**Decision:** Create a new endpoint.

**Rationale:** The existing `PUT /api/tour` is a full-update endpoint with 18 parameters and complex nested validation. Separating status updates into their own endpoint:
- Eliminates the bug without touching the full-update pipeline
- Reduces attack surface (fewer parameters to fuzz/validate)
- Follows RESTful semantics — PUT to a sub-resource is idiomatic
- Aligns with the existing pattern: `DELETE /api/tour/{id}/purge` already uses a sub-resource approach

**Alternative considered:** Modify the full-update endpoint to make `tourName`, `shortDescription`, `longDescription` nullable and validate conditionally. Rejected because it complicates the existing endpoint and makes the validator stateful.

### 2. JSON body (`[FromBody]`) vs FormData (`[FromForm]`)

**Decision:** JSON body with `[FromBody]`.

**Rationale:** The operation only needs one field (`status`). JSON is simpler to construct, easier to test, and produces cleaner logs than multipart boundaries. The existing `updateTourStatus` frontend code can be simplified from `FormData` construction to a plain object.

**Alternative considered:** `[FromForm]` with `application/x-www-form-urlencoded`. Rejected because the existing full-update endpoint already uses `[FromForm]` for file uploads — the `[FromBody]` distinction cleanly separates the two.

### 3. `ExecuteUpdateAsync` vs loading + saving entity

**Decision:** `ExecuteUpdateAsync` (EF Core 7+), with `WHERE` clause covering concurrency and soft-delete.

**Rationale:**
```
Load approach:   SELECT → deserialize → set field → UPDATE  (2 DB round trips)
ExecuteUpdate:  UPDATE WHERE id=? AND IsDeleted=? AND RowVersion=?  (1 DB round trip)
```
For a single-field update, `ExecuteUpdateAsync` is strictly better — it avoids the entity load entirely. The `LastModifiedBy` field (needed for audit) can be set in the same statement.

The `WHERE` clause must include:
- `IsDeleted == false` — prevents updating soft-deleted tours
- `RowVersion == @expectedVersion` — optimistic concurrency check; if 0 rows affected, return 409 Conflict

**Bypass of entity layer:** `ExecuteUpdateAsync` bypasses `TourEntity.Update()`. No domain logic in the entity method runs, no EF navigation properties are reloaded, and no domain events fire. This is intentional — consistent with the codebase's existing full-update path which also does not dispatch domain events. Documented explicitly in the repository implementation with a comment.

**Alternative considered:** Loading the entity first, updating in memory, saving. Rejected because it's slower, requires change tracking, and risks concurrency issues unnecessarily. If domain events become required later, this repository method must be replaced entirely with load-modify-save.

### 4. Cache invalidation via `ICacheInvalidator`

**Decision:** `UpdateTourStatusCommand` implements `ICacheInvalidator`.

**Rationale:** The existing `UpdateTourCommand` implements `ICacheInvalidator` with `CacheKeysToInvalidate => [CacheKey.Tour]`. The MediatR `CachingBehavior` pipeline fires cache eviction after the command succeeds. Using `ExecuteUpdateAsync` bypasses EF's change tracking, so the MediatR pipeline is the only cache invalidation mechanism. `UpdateTourStatusCommand` must implement `ICacheInvalidator` to ensure tour list caches are evicted on status change.

### 5. Validator for `UpdateTourStatusCommand`

**Decision:** Create `UpdateTourStatusCommandValidator` that checks `Status` is a valid enum.

**Rationale:** Consistent with the existing pipeline — FluentValidation runs via `ValidationBehavior`. Even though the controller could validate the enum at binding time, having a validator ensures the same rule applies if the command is triggered from other sources (MediatR directly, future CLI tools).

**Note on response shapes:** ASP.NET model binding throws `BadHttpRequestException` for out-of-range enum values (e.g., `status: 999`) BEFORE the request reaches FluentValidation. The error response for out-of-range integers is a raw 400 from model binding, not the `ValidationProblemDetails` format from FluentValidation. This is consistent with the existing codebase but worth noting — the error shape differs between invalid JSON, missing field, and out-of-range enum value.

### 6. Repository method `UpdateStatus` vs service-layer-only approach

**Decision:** Add `UpdateStatus` to `ITourRepository` + `TourRepository`.

**Rationale:** Keeps the service thin and consistent with the existing repository pattern. The service calls `tourRepository.UpdateStatus(id, status, userId)` — same pattern as other repository methods. Using `ExecuteUpdateAsync` keeps the repository implementation simple.

## API Contract

```
PUT /api/tour/{id}/status
Content-Type: application/json
Authorization: Bearer <token>
{
  "status": 2
}

Response 200:
{
  "success": true,
  "message": "Status updated successfully"
}

Response 404 (tour not found or soft-deleted):
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Tour not found."
}

Response 400 (invalid enum or malformed JSON):
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Status": ["Status must be a valid TourStatus value."]
  }
}

Response 409 (concurrent modification — RowVersion mismatch):
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.4",
  "title": "Conflict",
  "status": 409,
  "detail": "The tour was modified by another request. Please refresh and try again."
}
```

## Data Flow

```
Frontend                          Backend
   │                                  │
   │ PUT /api/tour/{id}/status       │
   │ { status: 2 }                    │
   ├─────────────────────────────────► TourController.UpdateStatus(id, status)
   │                                  │   ↓
   │                                  │ UpdateTourStatusCommand(id, status)
   │                                  │   ↓
   │                                  │ ValidationBehavior
   │                                  │   ↓ (passes)
   │                                  │ UpdateTourStatusCommandHandler
   │                                  │   ↓
   │                                  │ ITourService.UpdateStatus()
   │                                  │   ↓
   │                                  │ ITourRepository.UpdateStatus()
   │                                  │   ↓
   │                                  │ ExecuteUpdateAsync (SQL: UPDATE ... SET Status=?, LastModifiedBy=?, LastModifiedOnUtc=? WHERE Id=?)
   │                                  │   ↓
   │ 200 OK                          │
   │◄─────────────────────────────────┤
```

## File Changes Summary

### New files

| File | Purpose |
|------|---------|
| `panthora_be/src/Application/Features/Tour/Commands/UpdateTourStatusCommand.cs` | Command record: `Id`, `Status` |
| `panthora_be/src/Application/Features/Tour/Commands/UpdateTourStatusCommandValidator.cs` | FluentValidation: ensure `Status` is a valid enum |
| `panthora_be/src/Application/Features/Tour/Commands/UpdateTourStatusCommandHandler.cs` | MediatR handler, delegates to service |

### Modified files

| File | Change |
|------|--------|
| `panthora_be/src/Application/Services/ITourService.cs` | Add `UpdateStatus(Guid id, TourStatus status)` |
| `panthora_be/src/Application/Services/TourService.cs` | Implement `UpdateStatus()` |
| `panthora_be/src/Infrastructure/Persistence/Repositories/ITourRepository.cs` | Add `UpdateStatus()` signature |
| `panthora_be/src/Infrastructure/Persistence/Repositories/TourRepository.cs` | Implement `UpdateStatus()` with `ExecuteUpdateAsync` |
| `panthora_be/src/Api/Controllers/TourController.cs` | Add `UpdateStatus` action |
| `panthora_be/src/Api/Endpoint/TourEndpoint.cs` | Add `Status = "{id:guid}/status"` |
| `panthora_be/src/Application/Contracts/Booking/BookingManagementDtos.cs` | Add `UpdateTourStatusRequestDto` |
| `pathora_frontend/src/api/services/tourService.ts` | Change `updateTourStatus` to JSON + new endpoint |

## Risks / Trade-offs

| Risk | Impact | Mitigation |
|------|--------|------------|
| Concurrent status updates (race condition) | Two admins updating same tour simultaneously — last write wins | `WHERE RowVersion = @expectedVersion` in `ExecuteUpdateAsync`; return 409 on 0 rows |
| `ExecuteUpdateAsync` bypasses cache invalidation | Stale tour list cache serves old status | `UpdateTourStatusCommand` implements `ICacheInvalidator`; MediatR pipeline evicts cache |
| `ExecuteUpdateAsync` bypasses entity layer | No domain events, no `TourEntity.Update()` logic | Intentional — documented with comment in repository; consistent with existing codebase |
| Soft-deleted tour status updated | Updating a deleted tour silently succeeds | `WHERE IsDeleted = false` in `ExecuteUpdateAsync`; returns 0 rows → 404 |
| Enum validation response shape inconsistency | Out-of-range int throws model binding error, not FluentValidation | Model binding throws before reaching validator; different error format — accepted, consistent with codebase |
| Frontend still has `FormData` approach cached in browser | Old requests may be cached | Not a server concern; frontend cache cleared on page reload |

## Open Questions

1. **Should we emit a domain event on status change?** The codebase has `Aggregate<TId>` with `DomainEvents` infrastructure, but the existing `TourService.Update()` also bypasses domain events. Adding one now would require replacing `ExecuteUpdateAsync` with load-modify-save. **Decision: No domain events for phase 1** — if events are needed later, the repository method must be rewritten.
2. **Should we validate status transitions?** (e.g., `Pending → Active` requires approval notes). No state machine exists. **Decision: No transition validation for phase 1** — accept any valid enum value. Define the tour status state machine in a separate spec.
3. **Should we support PATCH semantics?** The broader architectural debt is the absence of a partial-update pattern. This plan adds one dedicated endpoint. A future `PATCH /api/tour/{id}` with JSON Merge Patch would address the systemic issue. **Decision: Not in phase 1 scope** — track as follow-up work.
