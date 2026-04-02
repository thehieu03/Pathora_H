# fix-backend-hardcodes

## Why

Backend codebase có 40+ chỗ hardcode values rải khắp nơi — từ magic numbers trong business logic đến string literals trùng lặp. Việc này gây khó bảo trì, dễ bug (như OutboxMessageTypes không khớp), và cần config để tune theo môi trường. Cần cleanup triệt để trước khi mở rộng feature.

## What Changes

- **Centralize magic numbers** vào Options classes hoặc Constants — giá trị có thể override qua config mà không cần sửa code
- **Thống nhất duplicate string literals** — đặc biệt là OutboxMessageTypes, SupportedLanguages, status strings
- **Fix bug OutboxMessageTypes** — 3 file cùng định nghĩa cùng message type bằng 3 string khác nhau (`PaymentCheck`, `SepayPaymentCheck`, `TourMediaCleanup` vs `SepayPaymentCheck` trong worker)
- **Extract shared constants** cho language codes, provider names, role names
- **Tạo centralized status-label mapping** — tránh hardcode string labels rải trong query results

### Breaking Changes

- Không có breaking change cho API hay data model
- Outbox worker message type strings sẽ thay đổi → **Outbox messages đang pending trong DB sẽ không được process** (migration cần chạy cleanup)

## Capabilities

### New Capabilities

- `backend-hardcode-cleanup`: Quản lý tất cả magic numbers, hardcoded strings, và duplicate definitions trong backend. Không tạo spec chi tiết vì đây là cleanup refactor, không thay đổi behavior.

### Modified Capabilities

- *(none — không thay đổi requirement nào của spec hiện tại)*

## Impact

### Thay đổi code

| Layer | Files affected |
|-------|----------------|
| Application | `IdentityService.cs`, `PaymentService.cs`, `PayOSClient.cs` |
| Infrastructure | `DependencyInjection.cs`, `JwtOptions.cs`, `AdminDashboardRepository.cs`, `LogProcessor.cs`, `CloudinaryCloudService.cs`, `SeaweedClient.cs`, `OutboxWorkerService.cs`, `PaymentProcessor.cs`, `MailDependencyInjection.cs` |
| Domain | `TourEntity.cs` |
| Api | `DependencyInjection.cs` (rate limiter), `LanguageResolutionMiddleware.cs` |
| Application/Common | `PublicLanguageResolver.cs`, `DefaultRoleIds.cs` |
| Repositories | `TourPurgeExecutor.cs` |

### Không ảnh hưởng

- Frontend (pathora/)
- API contracts / Swagger schema
- Database schema
- appsettings.json (credentials — xử lý riêng)
