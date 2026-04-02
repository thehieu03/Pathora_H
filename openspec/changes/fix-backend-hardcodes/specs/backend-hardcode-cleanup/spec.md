# backend-hardcode-cleanup

Specification for backend hardcode cleanup — refactoring magic numbers and duplicate string literals into centralized constants and options. **All requirements preserve existing behavior; no functional changes.**

## ADDED Requirements

### Requirement: All magic numbers shall be centralized into Options classes or constants

The system SHALL replace inline magic numbers throughout the backend with named constants in Options classes or static classes. Default values SHALL remain identical to the current hardcoded values to preserve behavior. Each magic number category SHALL be placed in its appropriate domain Options class.

#### Scenario: Database operation timeouts are configurable
- **WHEN** a database operation executes
- **THEN** the command timeout SHALL use the value from DatabaseOptions.CommandTimeoutSeconds (default: 120 seconds)
- **AND** the retry count SHALL use the value from DatabaseOptions.MaxRetryCount (default: 3)

#### Scenario: Cache duration is configurable
- **WHEN** a cache entry is created without explicit expiration
- **THEN** the expiration SHALL use the value from CacheOptions.DefaultExpirationMinutes (default: 5 minutes)

#### Scenario: JWT token lifetimes are configurable
- **WHEN** JWT tokens are generated
- **THEN** access token expiry SHALL use JwtOptions.ExpireInMinutes (default: 15 minutes)
- **AND** refresh token expiry SHALL use JwtOptions.RefreshTokenExpirationHours (default: 168 hours)
- **AND** password reset token expiry SHALL use JwtOptions.PasswordResetTokenExpirationMinutes (default: 15 minutes)

#### Scenario: Payment transaction expiration is configurable
- **WHEN** a payment transaction is created
- **THEN** the expiration SHALL use PaymentOptions.TransactionExpirationMinutes (default: 30 minutes)
- **AND** when PayOS returns an invalid expiry, the fallback SHALL use PaymentOptions.FallbackExpirationMinutes (default: 15 minutes)

#### Scenario: Mail service retry behavior is configurable
- **WHEN** a mail send fails
- **THEN** the retry count SHALL use MailOptions.MaxRetryAttempts (default: 3)
- **AND** the maximum delay between retries SHALL use MailOptions.MaxRetryDelayMs (default: 1500 milliseconds)
- **AND** the HTTP timeout SHALL use MailOptions.TimeoutSeconds (default: 30 seconds)

#### Scenario: Storage HTTP timeout is configurable
- **WHEN** a file upload or download is initiated via SeaweedFS
- **THEN** the HTTP client timeout SHALL use StorageOptions.HttpTimeoutSeconds (default: 30 seconds)

#### Scenario: Rate limiting is configurable
- **WHEN** a request is rate-limited
- **THEN** the global permit limit SHALL use RateLimitOptions.GlobalPermitLimit (default: 100 per minute)
- **AND** the auth endpoint permit limit SHALL use RateLimitOptions.AuthPermitLimit (default: 20 per minute)

#### Scenario: Admin dashboard parameters are configurable
- **WHEN** the admin dashboard loads statistics
- **THEN** the revenue month window SHALL use AdminDashboardOptions.DashboardMonthWindow (default: 12 months)
- **AND** the customer growth window SHALL use AdminDashboardOptions.CustomerGrowthMonthWindow (default: 6 months)
- **AND** the ranked items limit SHALL use AdminDashboardOptions.RankedItemLimit (default: 5)
- **AND** the nearly-full cancellation threshold SHALL use AdminDashboardOptions.NearlyFullCancellationThreshold (default: 90%)
- **AND** the danger cancellation rate threshold SHALL use AdminDashboardOptions.DangerCancellationRateThreshold (default: 5%)

#### Scenario: Tour code generation parameters are configurable
- **WHEN** a tour entity is created
- **THEN** the maximum code sequence value SHALL use TourOptions.CodeSequenceMaxValue (default: 100000)

### Requirement: Supported language codes shall be defined in a single location

The system SHALL define all supported language codes in `SupportedLanguageConstants` within the Application layer. Every code reference throughout the backend SHALL use these constants.

#### Scenario: Language resolver uses centralized language list
- **WHEN** a public language resolver checks if a language is supported
- **THEN** it SHALL use SupportedLanguageConstants.All which contains ["vi", "en"]

#### Scenario: Middleware uses centralized language list
- **WHEN** the language resolution middleware validates a language code
- **THEN** it SHALL use SupportedLanguageConstants.All

#### Scenario: Default language fallback uses centralized constant
- **WHEN** a user has no preferred language set
- **THEN** the system SHALL default to SupportedLanguageConstants.Vietnamese ("vi")

### Requirement: Outbox message types shall use a single source of truth

The system SHALL define all outbox message type strings in `OutboxMessageTypeConstants`. Every producer and consumer SHALL reference these constants, ensuring consistent message type names across the entire outbox pipeline.

#### Scenario: Payment check message type is consistent
- **WHEN** a payment check message is produced or consumed
- **THEN** the type string SHALL be OutboxMessageTypeConstants.PaymentCheck ("SepayPaymentCheck")

#### Scenario: Tour media cleanup message type is consistent
- **WHEN** a tour media cleanup message is produced or consumed
- **THEN** the type string SHALL be OutboxMessageTypeConstants.TourMediaCleanup ("TourMediaCleanup")

### Requirement: Auth provider names shall be defined as constants

The system SHALL define auth provider strings in `AuthProviderConstants`. Every reference to provider names SHALL use these constants.

#### Scenario: Google OAuth provider uses constant
- **WHEN** a user entity records Google OAuth creation
- **THEN** the provider string SHALL be AuthProviderConstants.Google ("google")

### Requirement: Email subjects shall be defined as constants

The system SHALL define email subject strings in `MailSubjectConstants`. Every email sending operation SHALL use these constants.

#### Scenario: Welcome email uses constant subject
- **WHEN** a welcome email is sent to a new user
- **THEN** the subject SHALL be MailSubjectConstants.Welcome

#### Scenario: Password reset email uses constant subject
- **WHEN** a password reset email is sent
- **THEN** the subject SHALL be MailSubjectConstants.PasswordReset

### Requirement: Tour status label strings shall be centralized

The system SHALL define all tour status display labels in `TourStatusLabelConstants`. Every repository generating status reports SHALL use these constants.

#### Scenario: Admin dashboard uses centralized status labels
- **WHEN** the admin dashboard generates category metrics
- **THEN** each status label SHALL be sourced from TourStatusLabelConstants
- **AND** the labels SHALL match: Pending, Confirmed, Deposited, Paid, Completed, Cancelled
