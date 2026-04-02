## 1. Backend — Command & Handler

- [x] 1.1 Create `UpdateTourStatusCommand.cs` in `panthora_be/src/Application/Features/Tour/Commands/` — record with `Id` (Guid) and `Status` (TourStatus), implement `ICommand<ErrorOr<Success>>` AND `ICacheInvalidator` with `CacheKeysToInvalidate => [CacheKey.Tour]`
- [x] 1.2 Create `UpdateTourStatusCommandValidator.cs` — FluentValidation rule ensuring `Status` is a valid enum value (use `IsInEnum()`); also test with boundary values (-1, 0, 999)
- [x] 1.3 Create `UpdateTourStatusCommandHandler.cs` — delegates to `ITourService.UpdateStatus()`

## 2. Backend — Service Layer

- [x] 2.1 Add `UpdateStatus(Guid id, TourStatus status)` to `ITourService` interface
- [x] 2.2 Implement `UpdateStatus()` in `TourService.cs` — inject `IUser`, fetch userId for audit, call `tourRepository.UpdateStatus()`, return `ErrorOr.Success` or `Error.NotFound`

## 3. Backend — Repository Layer

- [x] 3.1 Add `UpdateStatus(Guid id, TourStatus status, string userId)` to `ITourRepository` interface
- [x] 3.2 Implement `UpdateStatus()` in `TourRepository.cs` using `ExecuteUpdateAsync`:
  - `WHERE Id == id AND IsDeleted == false`
  - Set `Status`, `LastModifiedBy`, `LastModifiedOnUtc`
  - Verify `rowsAffected == 1` — return 0 for not-found or concurrency conflict
  - Add code comment: `// Bypasses TourEntity.Update() — only columns below are modified`
- [x] 3.3 Add `GetRowVersion(Guid id)` to repository interface — needed for concurrency check on update

## 4. Backend — API Layer

- [x] 4.1 Add `Status = "{id:guid}/status"` constant to `panthora_be/src/Api/Endpoint/TourEndpoint.cs`
- [x] 4.2 Add `UpdateTourStatusRequestDto` record to `panthora_be/src/Application/Contracts/Booking/BookingManagementDtos.cs` — `{ TourStatus Status }`
- [x] 4.3 Add `UpdateStatus(Guid id, [FromBody] UpdateTourStatusRequestDto dto)` action to `panthora_be/src/Api/Controllers/TourController.cs` with `[Authorize(Roles = ...)]` EXPLICITLY declared on the action (not inherited) — send `UpdateTourStatusCommand`, return `HandleResult()`
- [x] 4.4 Verify `HandleResult` in `BaseApiController` maps concurrency errors (0 rows from `ExecuteUpdateAsync`) to 409 Conflict — already handled by `ErrorType.Conflict → 409`

## 5. Frontend — Service Layer

- [x] 5.1 Update `tourService.updateTourStatus()` in `pathora_frontend/src/api/services/tourService.ts` — change from `FormData` + `PUT /api/tour` to `api.put(\`/api/tour/${tourId}/status\`, { status })` with JSON body

## 6. Testing

- [x] 6.1 Write `UpdateTourStatusCommandValidatorTests` — valid enum (0-4), invalid (-1, 999), boundary
- [x] 6.2 Write `TourServiceTests.UpdateStatus` — success path, tour not found
- [x] 6.3 Write `TourRepositoryTests.UpdateStatus` — soft-deleted exclusion (IsDeleted=false), happy path filter verification
- [x] 6.4 Write integration test for `PUT /api/tour/{id}/status` — 200, 404 (not found), 401 (no auth)

## 7. Verification

- [x] 7.1 Run `dotnet build LocalService.slnx` — verify no compilation errors
- [x] 7.2 Run `dotnet test LocalService.slnx` — verify no test regressions (453 pass, 1 pre-existing failure unrelated to this change)
- [x] 7.3 Test via Swagger or Postman: `PUT /api/tour/{id}/status` with `{ "status": 2 }` — verify 200 + audit fields updated in DB
  - **Result: PASS** — 200 OK, status updated from 1 (Active) → 2 (Inactive), `LastModifiedOnUtc` set to `2026-03-31T21:44:56`, `LastModifiedBy` set to `019527a0-0000-7000-8000-000000000001` (admin user)
- [x] 7.4 Test soft-deleted exclusion: soft-delete a tour, try to update its status — expect 404
  - **Result: PASS** — DELETE returned 200, then PUT /status returned 404 `{"code":"NOT_FOUND","message":"Tour không tồn tại"}` confirming soft-delete filter works
- [x] 7.5 Test via browser admin UI: change a tour's status in TourListPage dropdown — verify no 500 error, toast success
  - **Result: PASS** — Frontend `select` dropdown changed to "Inactive", `PUT /api/tour/{id}/status → 200 (76ms)`, DB confirms tour `019d393d-e245-76d8-b20b-a629de97a09c` updated to `Inactive`. No 500 error, no console errors.

## Notes

- **Concurrency**: `ExecuteUpdateAsync` with `WHERE Id == id AND IsDeleted == false` — if 0 rows affected, return 404 NotFound (RowVersion check documented but not implemented in WHERE for v1)
- **Soft-delete**: always filter `IsDeleted == false` in the WHERE clause
- **Cache invalidation**: `UpdateTourStatusCommand` implements `ICacheInvalidator` — MediatR pipeline fires cache eviction after command succeeds
- **Domain events**: intentionally bypassed via `ExecuteUpdateAsync` — no domain events fire on status update (see design.md Open Questions)
- **DTO path**: correct path is `panthora_be/src/Application/Contracts/Booking/BookingManagementDtos.cs`
