## Why

The tour admin dashboard has a bug when updating a tour's status: the frontend sends only `id` and `status` to the full update endpoint (`PUT /api/tour`), which requires `tourName`, `shortDescription`, and `longDescription` as non-nullable fields. This causes a `ValidationException` (500) and prevents admins from changing tour status. The issue is confirmed by `Content-Length: 263` — the request body contains only the two fields needed for a status change, not the full payload.

## What Changes

- **New endpoint**: `PUT /api/tour/{id}/status` — dedicated endpoint accepting only `{ status }`, replacing the current workaround that misuses the full update endpoint.
- **New command**: `UpdateTourStatusCommand` with `Id` + `Status` (2 fields only), implements `ICacheInvalidator` for cache eviction.
- **New service method**: `ITourService.UpdateStatus()` — thin wrapper using `ExecuteUpdateAsync` (single SQL UPDATE with optimistic concurrency check and soft-delete filter).
- **Frontend fix**: `tourService.updateTourStatus()` now calls the new endpoint with JSON body.
- **Backend validation**: `UpdateTourStatusCommandValidator` ensures `Status` is a valid enum value.
- **Optimistic concurrency**: `ExecuteUpdateAsync` uses `RowVersion` check in `WHERE` clause — returns 409 Conflict on concurrent modification.
- **Soft-delete protection**: Repository filters `IsDeleted == false` — cannot update deleted tours.
- **Cache invalidation**: `UpdateTourStatusCommand` implements `ICacheInvalidator` — MediatR pipeline evicts tour caches.
- **Audit trail**: `LastModifiedBy` and `LastModifiedOnUtc` are updated via repository.

## Capabilities

### New Capabilities

- `tour-status-update`: A dedicated API endpoint and service method for updating only a tour's status field. Reuses the existing `TourStatus` enum and repository infrastructure. Uses `ExecuteUpdateAsync` for performance.

### Modified Capabilities

<!-- No existing spec-level requirements change. The status update is a new capability added alongside the existing full-tour-update capability. -->

## Impact

### Backend (ASP.NET Core 10)

- New command: `Application/Features/Tour/Commands/UpdateTourStatusCommand.cs` — implements `ICacheInvalidator`
- New validator: `Application/Features/Tour/Commands/UpdateTourStatusCommandValidator.cs`
- New handler: `Application/Features/Tour/Commands/UpdateTourStatusCommandHandler.cs`
- New service interface method: `ITourService.UpdateStatus()`
- New service implementation: `TourService.UpdateStatus()`
- New repository method: `ITourRepository.UpdateStatus()` (using `ExecuteUpdateAsync` with `WHERE Id=? AND IsDeleted=? AND RowVersion=?`)
- Controller: new `UpdateStatus` action on `TourController` with explicit `[Authorize]`
- Endpoint: new route `PUT /api/tour/{id}/status`

### Frontend (Next.js 16)

- `api/services/tourService.ts`: change `updateTourStatus` from `FormData` + `/api/tour` to `api.put()` + `/api/tour/{id}/status` with JSON body
- No component changes needed — the existing status dropdown in `TourListPage` already calls `tourService.updateTourStatus()`
- No type changes needed — existing `TourStatus` enum maps directly

### No Breaking Changes

- The existing `PUT /api/tour` full update endpoint remains unchanged
- All existing consumers of the full update endpoint are unaffected
- No database schema changes
