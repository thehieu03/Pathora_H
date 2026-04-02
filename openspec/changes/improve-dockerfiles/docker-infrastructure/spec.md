# docker-infrastructure

## ADDED Requirements

### Requirement: Backend production Dockerfile builds and runs the API service

The `panthora_be/src/Api/Dockerfile` SHALL build the `Api.csproj` project (not Infrastructure.DatabaseMigration) and run `Api.dll` as the container entrypoint.

#### Scenario: Production build succeeds
- **WHEN** `docker build -f panthora_be/src/Api/Dockerfile` is executed from the `panthora_be/` context
- **THEN** dotnet restore, build, and publish SHALL execute against `src/Api/Api.csproj`
- **AND** the final image SHALL contain only the published API output
- **AND** `ENTRYPOINT ["dotnet", "Api.dll"]` SHALL be present in the final stage

#### Scenario: API health endpoint responds
- **WHEN** container starts with `docker run` and waits 30 seconds
- **THEN** `curl http://localhost:8080/health/live` SHALL return HTTP 200

#### Scenario: Non-root execution
- **WHEN** container runs as root in Docker daemon
- **THEN** the final stage SHALL switch to a non-root user (`appuser:appgroup`, UID 1000) before the ENTRYPOINT

### Requirement: All Dockerfiles use non-root users

All four Dockerfiles (backend prod, backend dev, frontend prod, frontend dev) SHALL execute their entrypoint processes as non-root users.

#### Scenario: Backend containers run as non-root
- **WHEN** backend container starts
- **THEN** the running process SHALL NOT have UID 0 (root)
- **AND** file ownership of `/app` SHALL be owned by `appuser:appgroup`

#### Scenario: Frontend containers run as non-root
- **WHEN** frontend container starts
- **THEN** the running process SHALL NOT have UID 0 (root)
- **AND** file ownership of `/app` SHALL be owned by `nextjs:nodejs` (UID 1001) or `node:node` (dev)

### Requirement: All Dockerfiles include healthchecks

All four Dockerfiles SHALL define a `HEALTHCHECK` directive that validates service availability.

#### Scenario: Backend production healthcheck
- **WHEN** `HEALTHCHECK` directive is evaluated by Docker daemon
- **THEN** it SHALL use `curl -f http://localhost:8080/health/live` or equivalent
- **AND** interval SHALL be 30s, timeout 10s, retries 3, start_period 30s

#### Scenario: Backend dev healthcheck
- **WHEN** `HEALTHCHECK` directive is evaluated
- **THEN** it SHALL use `curl -f http://localhost:8080/health/live`
- **AND** interval SHALL be 30s, timeout 10s, retries 3, start_period 30s

#### Scenario: Frontend production healthcheck
- **WHEN** `HEALTHCHECK` directive is evaluated
- **THEN** it SHALL use `wget --no-verbose --tries=1 --spider http://localhost:3001` (already present)
- **AND** interval SHALL be 30s, timeout 10s, retries 3, start_period 10s
- **AND** `wget` SHALL be available in the runner stage (via `apk add wget`)

#### Scenario: Frontend dev healthcheck
- **WHEN** `HEALTHCHECK` directive is added
- **THEN** it SHALL use `wget --no-verbose --tries=1 --spider http://localhost:3001`
- **AND** interval SHALL be 30s, timeout 10s, retries 3, start_period 15s

### Requirement: Backend layer caching optimization

The backend Dockerfiles SHALL order COPY instructions to maximize layer cache hits during rebuilds.

#### Scenario: Csproj restore layer is cached on source change
- **WHEN** only `.cs` source files change (no csproj changes)
- **THEN** Docker SHALL NOT re-execute `dotnet restore`
- **AND** Docker SHALL only re-execute `dotnet build` and `dotnet publish`

#### Scenario: Csproj restore layer is invalidated on dependency change
- **WHEN** any `.csproj` file changes
- **THEN** Docker SHALL re-execute `dotnet restore`
- **AND** subsequent layers SHALL be rebuilt

### Requirement: Frontend layer caching optimization

The frontend Dockerfiles SHALL order COPY instructions to maximize layer cache hits.

#### Scenario: Production: npm ci cached when package files unchanged
- **WHEN** only source files change (no `package.json` or `package-lock.json` changes)
- **THEN** Docker SHALL NOT re-execute `npm ci`
- **AND** the `node_modules` layer from `deps` stage SHALL be reused

#### Scenario: Dev: npm ci cached on source change
- **WHEN** only source files change
- **THEN** Docker SHALL NOT re-execute `npm ci`

### Requirement: Environment variable injection via docker-compose

Dockerfiles SHALL NOT hardcode environment-specific values. All runtime configuration SHALL be injectable via docker-compose or `docker run -e`.

#### Scenario: Backend API gateway URL
- **WHEN** container starts with `NEXT_PUBLIC_API_GATEWAY` env var set
- **THEN** the application SHALL use that value (not a hardcoded fallback)

#### Scenario: Frontend API gateway URL
- **WHEN** `NEXT_PUBLIC_API_GATEWAY` is passed as a build ARG
- **THEN** the Next.js app SHALL bake that value into the standalone build
- **AND** the runner stage SHALL use the same value as ENV fallback
