# Orma backend

Spring Boot 4.1.1 API (artifact orma-api), Java 21, Maven without wrapper (use `mvn`).
PostgreSQL with Flyway, JPA with `ddl-auto=validate` and `open-in-view=false`.
Security: OAuth2 resource server (RS256 JWT), Argon2id (`SecurityConfig`), rate limits with Bucket4j + Caffeine.
OpenAPI with springdoc (webmvc-api), enabled in the dev profile only.

## Package map (src/main/java/com/orma)

- `account`: controller dto mapper model repository service. `AccountActivity` (service) is a port
  owned by account and implemented by transaction (`TransactionAccountActivity`), so account never
  imports transaction.
- `auth`: controller dto mapper model repository service. Registration, sessions
  (`SessionService`, `RefreshTokenService`, `AccessTokenService`), email tokens, password reset,
  profile and preferences (`ProfileService`), current user (`CurrentUserService`).
  Emails: sealed `AuthEmail` records, sent by `AuthMailer` from a `@TransactionalEventListener`.
- `category`: controller dto mapper model repository service.
- `transaction`: controller dto mapper model repository service. Filters with JPA Specifications
  (`TransactionSpecifications`), totals in a repository fragment (`TransactionTotals`,
  `TransactionTotalsImpl`), CSV export (`TransactionCsv`).
- `dashboard`: controller dto service, no tables. Reads transactions, rates and the profile.
- `fx`: ECB rates from Frankfurter (`FrankfurterRateSource` behind `RateSource`),
  `ExchangeRateService`, daily `ExchangeRateSync`.
- `privacy`: controller dto service. JSON export of all user data.
- `common`: `error` (ApiException, GlobalExceptionHandler), `money` (Money, CurrencyCode),
  `period` (PayCycle, PayDayShift, FinancialMonth), `persistence` (BaseEntity), `ratelimit`
  (RateLimiter, RateLimitPolicy), `security` (Principals, SecureTokens), `web` (PageResponse),
  `events`, `i18n` (Languages).
- `config`: SecurityConfig, JwtConfig, JwtProperties, OpenApiConfig, LocaleConfig, AsyncConfig,
  AppProperties.

## Conventions

- Controllers are thin: user id from `Principals.userId(jwt)`, then one service call.
- Every repository query takes the user id; another user's data answers 404.
- Errors: `new ApiException(status, code, args)` or `ApiException.invalidField(field, key, args)`.
  Texts in `messages.properties` and `messages_fr.properties`; `MessagesTest` fails when a key is
  missing in one of them.
- Lists return `PageResponse<T>` (items, page, size, totalItems, totalPages).
- Money: `BigDecimal`, `Money.parse`, `Money.format`, `Money.PATTERN` for DTO validation.
- DTOs are records with Bean Validation and `@Schema`. Name operations with
  `@Operation(operationId = ...)`: the frontend client is generated from them.
- Entities extend `BaseEntity` (UUID id, `@Version`, createdAt, updatedAt).
- Spring Boot 4 has no `RestClient.Builder` bean: use `RestClient.builder()` (see FrankfurterRateSource).
- Format: Spotless with google-java-format.

## Database

- Migrations in `src/main/resources/db/migration`, `V<n>__<description>.sql`, latest
  `V9__pay_cycle.sql`. Never edit an applied migration: add the next number.
- `db/dev/R__dev_user.sql` runs in the dev profile only (admin / Orma-dev-2026).
- `ddl-auto=validate`: entity field types must match the column types exactly.

## Profiles

- `dev`: imports `../.env` or `.env`, springdoc on, JWT keys generated when the key files are missing.
- `prod`: needs `APP_BASE_URL`, `MAIL_FROM`, `JWT_PRIVATE_KEY_PATH`, `JWT_PUBLIC_KEY_PATH`.
- App settings are the `app.*` keys in `application.properties` (auth TTLs, lockout, fx).

## Tests

- Unit tests only: JUnit 5, Mockito, AssertJ (ADR 0004). No Testcontainers, no Spring context.
- `src/test/java/com/orma` mirrors the main packages.
- A stub shared by tests that do not all use it needs `lenient()`.

## Commands (run in backend/)

- Compile check: `mvn -q compile`
- Tests: `mvn test`, one class: `mvn test -Dtest=ClassName`
- Format: `mvn spotless:apply`. CI runs `mvn -B -ntp spotless:check` then `mvn -B -ntp verify`.
- Run: `docker compose up -d` at the root, then `mvn spring-boot:run -Dspring-boot.run.profiles=dev`
