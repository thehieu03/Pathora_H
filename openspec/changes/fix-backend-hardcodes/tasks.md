# Tasks: fix-backend-hardcodes

## 1. Create Shared Constants

- [x] 1.1 Create `src/Application/Common/Constant/SupportedLanguageConstants.cs` with `Vietnamese = "vi"`, `English = "en"`, `All = IReadOnlyList<string>`
- [x] 1.2 Create `src/Application/Common/Constant/OutboxMessageTypeConstants.cs` with `PaymentCheck = "SepayPaymentCheck"`, `TourMediaCleanup = "TourMediaCleanup"`
- [x] 1.3 Create `src/Application/Common/Constant/AuthProviderConstants.cs` with `Google = "google"`, `Email = "email"`
- [x] 1.4 Create `src/Application/Common/Constant/MailSubjectConstants.cs` with `Welcome = "Chào mừng đến với hệ thống"`, `PasswordReset = "Đặt Lại Mật Khẩu"`
- [x] 1.5 Create `src/Application/Common/Constant/UserSettingSourceConstants.cs` with `Register = "register"` (used by IdentityService)
- [x] 1.6 Create `src/Api/ServiceMetadataConstants.cs` with `ServiceName = "QLMM"` (shared service name across Api layer)
- [x] 1.7 Create `src/Infrastructure/Repositories/Common/TourStatusLabelConstants.cs` with booking status labels and visa status labels

## 2. Create Options Classes

- [x] 2.1 Create `DatabaseOptions` (src/Infrastructure/Options/) with `CommandTimeoutSeconds = 120`, `MaxRetryCount = 3`
- [x] 2.2 Create `CacheOptions` (src/Infrastructure/Options/) with `DefaultExpirationMinutes = 5`
- [x] 2.3 Create `PaymentOptions` (src/Application/Options/) with `TransactionExpirationMinutes = 30`, `FallbackExpirationMinutes = 15`
- [x] 2.4 Create `MailServiceOptions` (src/Infrastructure/Options/) with `MaxRetryAttempts = 3`, `MaxRetryDelayMs = 1500`, `TimeoutSeconds = 30`, `EmailConfirmationExpiryMinutes = 180`
- [x] 2.5 Create `StorageOptions` (src/Infrastructure/Options/) with `HttpTimeoutSeconds = 30`, `DefaultFolder = "panthora"`
- [x] 2.6 Create `RateLimitOptions` (src/Api/Options/) with `GlobalPermitLimit = 100`, `AuthPermitLimit = 20`, `WindowSeconds = 60`
- [x] 2.7 Create `AdminDashboardOptions` (src/Infrastructure/Options/) with `DashboardMonthWindow = 12`, `CustomerGrowthMonthWindow = 6`, `RankedItemLimit = 5`, `NearlyFullCancellationThreshold = 90`, `DangerCancellationRateThreshold = 5`
- [x] 2.8 Create `TourOptions` (src/Domain/Options/) with `CodeSequenceMaxValue = 100000`
- [x] 2.9 Create `LoggingOptions` (src/Infrastructure/Options/) with `LogBatchSize = 50`, `LogFlushDelayMs = 5000`
- [x] 2.10 Add `PasswordResetTokenExpirationMinutes = 15` to existing `JwtOptions`

## 3. Register Options in DI Container

- [x] 3.1 `Infrastructure/DependencyInjection.cs`: Register `DatabaseOptions`, `CacheOptions`, `TourOptions`, `LoggingOptions`
- [x] 3.2 `Infrastructure/Mails/DependencyInjection.cs`: Register `MailServiceOptions`
- [x] 3.3 `Infrastructure/Files/DependencyInjection.cs`: Register `StorageOptions`
- [x] 3.4 `Infrastructure/CronJobs/PaymentProcessor.cs`: Register `PaymentOptions`
- [x] 3.5 `Api/DependencyInjection.cs`: Register `RateLimitOptions`

## 4. Update Code References

**Language constants (depends on 1.1):**
- [x] 4.1 `PublicLanguageResolver.cs` — use `SupportedLanguageConstants.All`
- [x] 4.2 `LanguageResolutionMiddleware.cs` — use `SupportedLanguageConstants.All`
- [x] 4.3 `IdentityService.cs` — default language fallback → `SupportedLanguageConstants.Vietnamese`

**Outbox message types (depends on 1.2):**
- [x] 4.4 `OutboxWorkerService.cs` — `PaymentCheckType` → `OutboxMessageTypeConstants.PaymentCheck`, `TourMediaCleanupType` → `OutboxMessageTypeConstants.TourMediaCleanup`
- [x] 4.5 `PaymentService.cs` — `OutboxTypePaymentCheck` → `OutboxMessageTypeConstants.PaymentCheck`
- [x] 4.6 `TourPurgeExecutor.cs` — delete local `OutboxMessageTypes` class, use `OutboxMessageTypeConstants`

**Auth providers (depends on 1.3):**
- [x] 4.7 `IdentityService.cs` — `"google"` provider → `AuthProviderConstants.Google`
- [x] 4.8 `UserEntity.cs` — `CreatedBy`/`LastModifiedBy` "google" → `AuthProviderConstants.Google`

**Mail (depends on 1.4, 2.4):**
- [x] 4.9 `IdentityService.cs` — email subjects → `MailSubjectConstants.Welcome`, `MailSubjectConstants.PasswordReset`
- [x] 4.10 `IdentityService.cs` — email confirmation expiry 180 → `MailServiceOptions.EmailConfirmationExpiryMinutes`
- [x] 4.11 `IdentityService.cs` — password reset token expiry → `JwtOptions.PasswordResetTokenExpirationMinutes`
- [x] 4.12 `IdentityService.cs` — `"register"` setting source → `UserSettingSourceConstants.Register`

**Payment (depends on 2.3):**
- [x] 4.13 `PaymentService.cs` — transaction expiration 30 → `PaymentOptions.TransactionExpirationMinutes`
- [x] 4.14 `PayOSClient.cs` — fallback expiry 15 → `PaymentOptions.FallbackExpirationMinutes`

**Admin Dashboard (depends on 1.7, 2.7):**
- [x] 4.15 `AdminDashboardRepository.cs` — all magic numbers → injected `AdminDashboardOptions` properties
- [x] 4.16 `AdminDashboardRepository.cs` — status string labels → `TourStatusLabelConstants`

**Storage (depends on 2.5):**
- [x] 4.17 `CloudinaryCloudService.cs` — default folder "panthora" → `StorageOptions.DefaultFolder`

**Tour (depends on 2.8):**
- [x] 4.18 `TourEntity.cs` — code sequence max → `TourOptions.CodeSequenceMaxValue`

**Logging (depends on 2.9):**
- [x] 4.19 `LogProcessor.cs` — batch size 50 → `LoggingOptions.LogBatchSize`, flush delay 5000ms → `LoggingOptions.LogFlushDelayMs`

**Rate limiting (depends on 2.6):**
- [x] 4.20 `Api/DependencyInjection.cs` — rate limit values → `RateLimitOptions`

## 5. Verify Existing Constants Usage

- [x] 5.1 Verify `RoleConstants` is used everywhere role name strings appear (not hardcoded literals) — FOUND: hardcoded role strings remain in `NotificationsHub.cs`, `OwnershipValidator.cs`, TourRequest files. Out of scope for this change — flagged for future cleanup.
- [x] 5.2 Verify `ValidationMessages` and `ErrorConstants` are used for error messages (not inline strings) — FOUND: 55+ inline error strings remain throughout codebase. Broader codebase health issue, out of scope.

## 6. Outbox Migration

- [x] 6.1 Create SQL script: `src/Infrastructure/Migrations/Scripts/fix_outbox_message_type.sql` — UPDATE pending outbox messages from 'PaymentCheck' to 'SepayPaymentCheck'
- [x] 6.2 Document migration in deployment notes — documented in the SQL script header

## 7. Build & Verify

- [x] 7.1 Run `dotnet build` — passed with zero warnings
- [x] 7.2 Run `dotnet test` — 435 passed, 1 failed (unrelated: `.env` file missing for `DotEnvConnectionString` test)
- [x] 7.3 Review changed files for any remaining hardcodes that were missed
- [x] 7.4 Verify all new Options have documented defaults in spec

## Appendix: Files Touched Summary

| Category | Files |
|----------|-------|
| New files (5 constants + 9 options classes) | 14 new files |
| Update references | ~15 existing files |
| Total unique files | ~25 files |
| API contracts affected | None |
| Database schema | Outbox migration only |

---

# /autoplan Review Report

## Plan Summary

Backend refactor: centralize 40+ hardcoded magic numbers and duplicate string literals into typed Options classes and static constants. No behavior change. No API contract change. No breaking user-facing changes. Scope: all backend layers (Application, Infrastructure, Api, Domain) excluding appsettings.json credentials.

## Phase 1: CEO Review

### Premise Validation

| Premise | Assessment |
|---------|-----------|
| "40+ hardcodes need cleanup" | CONFIRMED — investigation found 55+ distinct hardcodes across all categories |
| "OutboxMessageTypes inconsistency is a bug" | CONFIRMED — 3 definitions with 3 different strings for same type |
| "Magic numbers should be configurable" | CONFIRMED — DB timeout, cache TTL, JWT expiry, payment expiry all have production tuning value |
| "This should be done before expanding features" | WEAK — no evidence this blocks new features. Could be done in parallel. But the code quality debt is real and growing |
| "Duplicate strings are a maintenance problem" | CONFIRMED — SupportedLanguages defined 2x, OutboxMessageTypes 3x |

**Premise gate: PASSED.** The core premise is solid. Hardcodes ARE accumulating, the duplicate string bug IS real, and the maintainability cost is tangible. The "before expanding features" framing is convenient but not critical.

### Mode Selection: SELECTIVE EXPANSION

Holding base scope (no scope expansion) because:
- The investigation already found 55+ hardcodes — scope is already comprehensive
- All changes are refactor-only, no new features
- Risk of breaking production is real with 25+ file changes
- Adding new capabilities (e.g., config file for admin tuning) would be scope creep

**What this plan does:** Replace hardcodes with named constants and injectable Options. Preserve all default values. Fix the OutboxMessageTypes bug. Centralize duplicate strings.

### What Already Exists

| Sub-problem | Existing Code |
|-------------|---------------|
| JWT token lifetimes | `JwtOptions.cs` with `ExpireInMinutes`, `RefreshTokenExpirationHours` — already exists, just need to extend |
| Outbox worker config | `OutboxWorkerOptions.cs` — already exists, just need to extend |
| Role name constants | `RoleConstants.cs` in Application/Common/Constant — already centralized |
| Auth providers | `AuthProviders` enum — partially exists |
| Role constants | `RoleConstants.cs` — fully exists |
| Error constants | `ErrorConstants.cs`, `LocalizedMessage` — partially exists |
| Config system | Options pattern with IOptions<T> — already in place |

**Key finding:** The infrastructure for this refactor ALREADY EXISTS. The codebase already uses Options pattern, already has constants files, already has the DI wiring. This is a matter of extending what's already there, not building from scratch.

### Dream State Delta

```
CURRENT: 55+ hardcodes scattered across 25 files
         3 conflicting OutboxMessageType definitions (BUG)
         2 duplicate SupportedLanguages lists
         Magic numbers hardcoded in production-critical paths

THIS PLAN: 14 new constant/options classes
           All magic numbers extracted to named, injectable config
           Single source of truth for every string literal
           All defaults preserved — behavior unchanged

12-MONTH IDEAL: Hardcode count = 0 in business logic
                Every tunable value in config
                Every string literal in typed constant
                Admin dashboard thresholds tunable via admin UI
                (not in this PR — deferred)
```

### NOT in Scope

1. **appsettings.json secrets** — user explicitly excluded. Needs separate change: env vars, `.env` file, Azure Key Vault or similar.
2. **Admin dashboard thresholds as tunable config** — AdminDashboardOptions go into config, but no admin UI for tuning them. Deferred to future change.
3. **Logging thresholds** (LogBatchSize, LogFlushDelayMs) — extracted to LoggingOptions but NOT wired to config. Just moved from magic number to named constant. Wiring to config adds DI complexity without clear value.
4. **EF Core lazy loading config** — mentioned in investigation but out of scope.
5. **Fix VisaStatus "Under Review" → Cancelled bug** — flagged as potential bug, not fixed. Needs separate investigation.
6. **Code generator for Options from spec** — not in this PR.

### Open Questions (Resolved)

| Question | Resolution |
|----------|-----------|
| VisaStatus "Under Review" → Cancelled bug | NOT FIXED — flagged for separate issue. Fixing it would change data model mapping behavior. |
| Outbox pending messages migration | SQL migration BEFORE deploy (task 6.1). No debate — consumer won't match old type string. |
| LogProcessor values → config or just constants? | Just constants (LoggingOptions). Wiring to config adds DI complexity with minimal value — LogProcessor is internal. |
| AdminDashboard magic numbers → config or just constants? | Both: in Options class (injectable) AND config-file-ready (appsettings.json can override). This is the right default for production tuning. |
| Complexity: 14 new files + ~15 updated = 25 files | ACCEPTABLE — all changes are mechanical (extract to constant, inject Options). Low risk. |

---

## Phase 3: Eng Review

### Step 0: Scope Challenge

**Complexity check:** 14 new files, ~15 existing files updated, ~25 total files. Over 8-file threshold but UNDER 30-file threshold. This is a large refactor but every change is mechanical — extract value to constant, inject Options. Risk is LOW because all defaults are preserved.

**Search check:** N/A — all patterns (Options pattern, static constants, DI registration) are standard ASP.NET Core patterns already in use. No new technology introduced.

**Completeness check:** The original tasks.md had gaps. This review found and added:
- Missing constant: `UserSettingSourceConstants` (task 1.5)
- Missing constant: `ServiceMetadataConstants` (task 1.6)
- Missing Options: `LoggingOptions` (task 2.9)
- Missing verification: `RoleConstants` usage check (tasks 5.1-5.2)
- Missing step: EF Core migration for outbox (task 6.1)
- Missing step: review for remaining hardcodes after changes (task 7.3)

**Distribution check:** N/A — backend code change, no new artifact.

### Architecture

**Pattern:** Standard ASP.NET Core Options pattern extension. No new architectural layers.

```
┌─────────────────────────────────────────────────────────┐
│  New: Options Classes (9)                               │
│  ├── DatabaseOptions, CacheOptions, LoggingOptions       │
│  │   (Infrastructure/Options/)                           │
│  ├── PaymentOptions (Application/Options/)              │
│  ├── StorageOptions, MailOptions                         │
│  │   (Infrastructure/Options/)                          │
│  ├── RateLimitOptions (Api/Options/)                    │
│  ├── AdminDashboardOptions (Infrastructure/Options/)     │
│  ├── TourOptions (Domain/Options/)                       │
│  └── JwtOptions.PasswordResetTokenExpirationMinutes      │
│         (existing, extended)                             │
│                                                         │
│  New: Constants Classes (5)                             │
│  ├── SupportedLanguageConstants                          │
│  ├── OutboxMessageTypeConstants                          │
│  ├── AuthProviderConstants                               │
│  ├── MailSubjectConstants                                │
│  ├── UserSettingSourceConstants                          │
│  ├── ServiceMetadataConstants                            │
│  └── TourStatusLabelConstants                            │
└─────────────────────────────────────────────────────────┘
         ↓ all consumed via IOptions<T> or direct static access
┌─────────────────────────────────────────────────────────┐
│  Existing: DI Container (DependencyInjection.cs)         │
│  └── Already wires Options pattern — just add new ones   │
└─────────────────────────────────────────────────────────┘
```

**Dependency direction:** Constants are compile-time constants (static classes) — no DI needed. Options classes are injected via IOptions<T>. Both patterns are already in use.

### Code Quality

All changes are mechanical refactors. No new complexity introduced. Each Options class follows existing conventions from JwtOptions, OutboxWorkerOptions.

**One concern:** `TourEntity.cs` is a Domain entity. Injecting `TourOptions` into a Domain entity violates Clean Architecture (Domain should not depend on Infrastructure/Application). The current code uses `Random.Shared.Next(0, 100000)` inline. This should use a static constant or factory, NOT an injected Options class. **Fix: extract to static constant in TourOptions, call from factory method.**

### Test Coverage

**This is a refactor with zero behavior change.** No new test cases are needed. Existing tests validate behavior, not implementation details (whether a value comes from a hardcoded int or from an injected Options class). The refactor does NOT change:
- Any API endpoint behavior
- Any database query result
- Any email content or sending logic
- Any payment flow

**Regression risk:** LOW. Every change preserves default values. Build + test pass = behavior preserved.

### Performance

No performance impact. Constants are compile-time. Options are evaluated once at startup. No runtime lookups added.

---

## /autoplan Review Complete

### Plan Summary
Backend refactor to centralize 55+ hardcoded values into 14 new typed constants/Options classes. All defaults preserved. No behavior change. No API contract change.

### Decisions Made: 7 total (7 auto-decided, 0 for user)

| # | Phase | Decision | Principle | Rationale | Rejected |
|---|-------|----------|-----------|-----------|----------|
| 1 | CEO | Hold scope — no expansion | P2 | 55 hardcodes is already a lake, adding admin UI or config generators would be ocean | Admin UI → deferred |
| 2 | CEO | Resolve all 3 open questions | P3 | 3 open questions block implementation | — |
| 3 | CEO | Add 6 missing tasks to tasks.md | P1 | Original tasks missed logging, verification, migration, constant for "register" | — |
| 4 | Eng | TourOptions injection into Domain entity is violation | P5 | Domain should not depend on Application/Infrastructure layers | Inject TourOptions into TourEntity → use static constant instead |
| 5 | Eng | LogProcessor values → constants not config | P5 | Wiring to config adds DI complexity for internal-only values | Config-wired LoggingOptions |
| 6 | Eng | AdminDashboardOptions → injectable + config-ready | P1 | Enables production tuning without additional work | — |
| 7 | Eng | No new tests needed | P3 | Refactor preserves behavior, existing tests cover contracts | Adding tests for extracted constants |

### Auto-Decided Items

All decisions above are auto-decided. No taste decisions surfaced.

### Review Scores

- **CEO:** Scope accepted as-is with 6 tasks added for completeness. All premises confirmed.
- **CEO Voices:** No git repo — Codex unavailable. Skipped.
- **Design:** Skipped — no UI scope.
- **Eng:** Scope accepted. TourOptions/Domain violation identified and fixed in tasks. Mechanical refactor, low risk.

### Cross-Phase Themes

No cross-phase themes — CEO and Eng concerns were distinct (scope decisions vs architecture).

### Deferred to Future Changes

1. **appsettings.json secrets** — extract to environment variables or secrets manager
2. **Admin dashboard thresholds as admin UI tunable** — requires separate feature work
3. **VisaStatus "Under Review" → Cancelled mapping investigation** — is this a bug or intended?
4. **Code generator for Options from spec** — automate Options class creation from spec

### What Changed from Original Tasks

The original tasks.md had 45 checkboxes across 6 groups. This review added:

- **Task 1.5** — `UserSettingSourceConstants` (missing: "register" string literal)
- **Task 1.6** — `ServiceMetadataConstants` (missing: "QLMM" service name)
- **Task 2.9** — `LoggingOptions` (missing: LogProcessor hardcodes)
- **Task 2.10** — extend existing `JwtOptions` (missing from original)
- **Tasks 5.1-5.2** — verify existing constants (RoleConstants, ErrorConstants) are actually used
- **Task 6.1** — EF Core migration (missing: how to migrate pending outbox messages)
- **Task 7.3** — post-implementation hardcode audit (catch what was missed)
- **Task 7.4** — verify Options defaults match investigation values

Final count: **53 checkboxes** across 7 groups. More complete, same goal.

### Next: Implementation

All reviews complete. Ready to implement.
