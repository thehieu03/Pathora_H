# Design: fix-backend-hardcodes

## Context

Backend codebase có 40+ hardcode values rải khắp nơi. Phần lớn nằm ngoài appsettings.json (đã loại trừ). Cần cleanup theo từng category để không break production.

**Current state:**
- Magic numbers: DB timeout, cache TTL, JWT expiry, payment expiry, mail retry, rate limits, batch sizes, thresholds
- Duplicate strings: SupportedLanguages định nghĩa 2 lần, OutboxMessageTypes định nghĩa 3 lần với 3 giá trị khác nhau
- String literals: Email subjects, provider names, role names, status labels
- Enum-as-strings: Roles, auth providers, outbox message types

## Goals / Non-Goals

**Goals:**
- Centralize tất cả magic numbers vào Options classes với default values hợp lý
- Thống nhất duplicate string definitions vào single source of truth
- Fix OutboxMessageTypes inconsistency bug
- Extract shared constants cho language codes, provider names
- Tất cả thay đổi giữ nguyên behavior — chỉ refactor, không đổi logic

**Non-Goals:**
- Không đụng appsettings.json (credentials, connection strings — xử lý riêng)
- Không đổi API contracts
- Không thay đổi database schema
- Không refactor config system hiện tại (Options pattern đã có, chỉ mở rộng)

## Decisions

### 1. Magic Numbers → Options Pattern (extend existing)

**Approach:** Thêm properties vào existing Options classes thay vì tạo class mới.

| Magic Number | Target Options Class | Property |
|---|---|---|
| DB CommandTimeout(120) | DatabaseOptions (mới) | CommandTimeoutSeconds |
| DB RetryOnFailure(3) | DatabaseOptions (mới) | MaxRetryCount |
| Cache Duration(5min) | CacheOptions (mới) | DefaultExpirationMinutes |
| JWT ExpireInMinutes(15) | JwtOptions | **existing** — giữ nguyên |
| JWT RefreshTokenExpirationHours(168) | JwtOptions | **existing** — giữ nguyên |
| JWT AccessTokenCookieExpirationHours(1) | JwtOptions | **existing** — giữ nguyên |
| Password reset token(15min) | JwtOptions | PasswordResetTokenExpirationMinutes |
| Payment ExpirationMinutes(30) | PaymentOptions (mới) | TransactionExpirationMinutes |
| PayOS fallback(15min) | PaymentOptions (mới) | FallbackExpirationMinutes |
| Mail MaxRetryAttempts(3) | MailOptions (mới) | MaxRetryAttempts |
| Mail MaxDelay(1500ms) | MailOptions (mới) | MaxRetryDelayMs |
| Mail Timeout(30s) | MailOptions (mới) | TimeoutSeconds |
| SeaweedFS Timeout(30s) | StorageOptions (mới) | HttpTimeoutSeconds |
| Rate Limit(100/min) | RateLimitOptions (mới) | GlobalPermitLimit |
| Rate Limit Auth(20/min) | RateLimitOptions (mới) | AuthPermitLimit |
| Outbox BatchSize(10) | OutboxWorkerOptions | **existing** — giữ nguyên |
| Outbox PollingInterval(30000ms) | OutboxWorkerOptions | **existing** — giữ nguyên |
| Outbox MaxRetries(10) | OutboxWorkerOptions | **existing** — giữ nguyên |
| Admin DashboardWindow(12) | AdminDashboardOptions (mới) | DashboardMonthWindow |
| CustomerGrowthWindow(6) | AdminDashboardOptions | CustomerGrowthMonthWindow |
| RankedItemLimit(5) | AdminDashboardOptions | RankedItemLimit |
| NearlyFullThreshold(90%) | AdminDashboardOptions | NearlyFullCancellationThreshold |
| DangerCancellationRate(5%) | AdminDashboardOptions | DangerCancellationRateThreshold |
| TourCodeSequenceMax(100000) | TourOptions (mới) | CodeSequenceMaxValue |

**Alternatives considered:**
- Tạo `HardcodeOptions` gom tất cả → rejected vì loãng, không phân loại theo domain
- Giữ magic numbers có comment → rejected vì vẫn hardcode, chỉ có thêm comment

### 2. OutboxMessageTypes → Single Source of Truth

**Approach:** Tạo `OutboxMessageTypeConstants` trong Application layer. Tất cả producer/consumer dùng cùng constant.

```csharp
// src/Application/Common/Constant/OutboxMessageTypeConstants.cs
public static class OutboxMessageTypeConstants
{
    public const string PaymentCheck = "SepayPaymentCheck";  // THỐNG NHẤT: dùng giá trị thực tế
    public const string TourMediaCleanup = "TourMediaCleanup";
}
```

Files cần update:
- `PaymentService.cs`: `OutboxTypePaymentCheck = OutboxMessageTypeConstants.PaymentCheck`
- `OutboxWorkerService.cs`: `PaymentCheckType = OutboxMessageTypeConstants.PaymentCheck`
- `TourPurgeExecutor.cs`: Xóa local class `OutboxMessageTypes`, dùng constant

**⚠️ Migration:** Messages đang pending trong outbox table với type `"PaymentCheck"` (value cũ) sẽ không match consumer. Cần:
1. SQL migration update: `UPDATE outbox_messages SET type = 'SepayPaymentCheck' WHERE type = 'PaymentCheck'`
2. Hoặc consumer match cả 2 strings trong transition period

### 3. SupportedLanguages → Single Source of Truth

**Approach:** Tạo `SupportedLanguageConstants` trong Application layer.

```csharp
// src/Application/Common/Constant/SupportedLanguageConstants.cs
public static class SupportedLanguageConstants
{
    public const string Vietnamese = "vi";
    public const string English = "en";
    public static readonly IReadOnlyList<string> All = [Vietnamese, English];
}
```

Files cần update:
- `PublicLanguageResolver.cs`: Dùng `SupportedLanguageConstants.All`
- `LanguageResolutionMiddleware.cs`: Dùng `SupportedLanguageConstants.All`
- `IdentityService.cs`: Dùng `SupportedLanguageConstants.Vietnamese` cho default

### 4. Status Labels → Centralized Mapping

**Approach:** Tạo `TourStatusLabelConstants` cho AdminDashboardRepository.

```csharp
// src/Infrastructure/Repositories/Common/TourStatusLabelConstants.cs
public static class TourStatusLabelConstants
{
    // Booking statuses
    public const string Pending = "Pending";
    public const string Confirmed = "Confirmed";
    public const string Deposited = "Deposited";
    public const string Paid = "Paid";
    public const string Completed = "Completed";
    public const string Cancelled = "Cancelled";

    // Visa statuses
    public const string VisaUnderReview = "Under Review";
    public const string VisaApproved = "Approved";
    public const string VisaRejected = "Rejected";
}
```

**⚠️ Bug found:** `"Under Review"` trong VisaStatus report đang map đến `TourRequestStatus.Cancelled`. Cần xác nhận đây là intended behavior hay bug — nếu bug thì cần fix mapping riêng.

### 5. Other Hardcoded Strings

| Item | Location | Action |
|---|---|---|
| Email subject "Chào mừng..." | IdentityService.cs:113 | Extract to `MailSubjectConstants` |
| Email subject "Đặt Lại Mật Khẩu" | IdentityService.cs:362 | Extract to `MailSubjectConstants` |
| Email confirmation expiry 180min | IdentityService.cs:114 | Add to JwtOptions or MailOptions |
| "google" provider | IdentityService.cs, UserEntity.cs | Use `AuthProvidersConstants.Google` |
| "register" setting source | IdentityService.cs:513 | Extract to `UserSettingSourceConstants` |
| Cloudinary default folder "panthora" | CloudinaryCloudService.cs:18 | Extract to `StorageConstants.DefaultFolder` |
| "QLMM" service name | DependencyInjection.cs:30 | Extract to `ServiceMetadata.ServiceName` |

### 6. AuthProvider Constants

Tạo `AuthProviderConstants` trong Application layer:

```csharp
// src/Application/Common/Constant/AuthProviderConstants.cs
public static class AuthProviderConstants
{
    public const string Google = "google";
    public const string Email = "email";
}
```

Update:
- `IdentityService.cs`: `"google"` → `AuthProviderConstants.Google`
- `UserEntity.cs`: `CreatedBy = "google"` → `AuthProviderConstants.Google`

## Risks / Trade-offs

| Risk | Severity | Mitigation |
|------|----------|------------|
| Outbox pending messages không match type mới | HIGH | SQL migration update type trước deploy, hoặc consumer match cả 2 strings |
| Behavior thay đổi vô tình khi refactor magic numbers | MEDIUM | Mỗi thay đổi giữ nguyên default value, chỉ extract ra constant/config |
| VisaStatus "Under Review" → Cancelled mapping là bug | MEDIUM | Không fix trong PR này, flag để tạo separate issue |
| Nhiều file thay đổi cùng lúc | LOW | Commit theo từng category (messages, languages, options...) |
| AdminDashboard magic numbers dùng trong raw SQL | LOW | Options class vẫn inject được, cần verify DI setup |

## Migration Plan

1. **Tạo all Constants classes** trước (Application/Common/Constant/)
2. **Update references** trong từng file
3. **Update Options classes** để inject magic numbers
4. **Build + test** sau mỗi category
5. **Outbox migration**: Chạy SQL update pending messages type trước khi deploy
6. **Verify**: `dotnet build` + `dotnet test`

Rollback: Revert git commit — tất cả thay đổi đều reversible.

## Open Questions

1. **VisaStatus "Under Review" → Cancelled**: Đây là intended behavior hay bug? Cần confirm với business logic.
2. **AdminDashboard magic numbers**: Có nên đưa vào config để admin có thể tune không, hay chỉ extract thành constants?
3. **Outbox pending messages**: Chạy migration SQL trước deploy hay deploy rồi mới chạy? Ưu tiên migration trước.
