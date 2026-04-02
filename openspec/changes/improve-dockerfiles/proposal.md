## Why

The existing Dockerfiles for both the backend (ASP.NET Core API) and frontend (Next.js) have multiple correctness and production-readiness issues. The backend production Dockerfile builds the wrong project (DatabaseMigration instead of API) and uses an incorrect runtime image. All four Dockerfiles lack proper non-root execution, healthchecks, and layer-caching optimization. These issues cause failed deployments, security risks (running as root), and slow CI/CD builds.

## What Changes

### Backend Production (`panthora_be/src/Api/Dockerfile`)
- Fix to build and run the **API project** (not DatabaseMigration)
- Use `mcr.microsoft.com/dotnet/aspnet:10.0` runtime image (not alpine runtime, which doesn't include ASP.NET Core hosting)
- Proper multi-stage build: restore → build → publish → runner
- Non-root user execution (appuser:appgroup, UID 1000)
- `libgssapi-krb5-2` dependency retained for Kerberos/GSSAPI support
- Healthcheck using `curl` against `/health` endpoint
- Expose port 8080 (HTTP) matching docker-compose
- Healthcheck interval: 30s, timeout: 10s, retries: 3, start_period: 30s

### Backend Dev (`panthora_be/src/Api/Dockerfile.dev`)
- Add non-root user execution (appuser:appgroup)
- Fix layer caching: restore all csproj files → restore → copy source → build → run
- Add `libgssapi-krb5-2` dependency
- Healthcheck using `curl` against `/health/live`
- All env vars passed via docker-compose, no hardcoding

### Frontend Production (`pathora/frontend/Dockerfile`)
- Fix healthcheck: install `wget` in runner stage OR switch to `curl`
- Use `ARG` + `ENV` pattern so `NEXT_PUBLIC_API_GATEWAY` can be overridden at build time
- Retain existing multi-stage build, standalone output, non-root user

### Frontend Dev (`pathora/frontend/Dockerfile.dev`)
- Add healthcheck using `wget` against port 3001
- Improve layer ordering: package files → npm ci → copy source → dev server
- All env vars passed via docker-compose

### Docker Compose Updates
- Backend: update healthcheck to use `curl -f http://localhost:8080/health/live` (not swagger)
- Frontend: healthcheck already uses wget correctly

## Capabilities

### New Capabilities
- `docker-infrastructure`: Docker build configuration for the Pathora monorepo. Defines multi-stage Dockerfile patterns, healthcheck conventions, non-root user practices, and layer-caching optimization for both ASP.NET Core backend and Next.js frontend services.

### Modified Capabilities
*(None — no existing specs change behavior)*

## Impact

### Code
- `panthora_be/src/Api/Dockerfile` — rewritten
- `panthora_be/src/Api/Dockerfile.dev` — improved
- `panthora_be/docker-compose.yml` — healthcheck update
- `pathora/frontend/Dockerfile` — healthcheck fix
- `pathora/frontend/Dockerfile.dev` — healthcheck + layer ordering

### Dependencies
- No new runtime dependencies
- Backend dev/prod still requires `libgssapi-krb5-2` for Kerberos (already present in prod Dockerfile, added to dev)

### Systems
- Docker build pipeline (CI/CD)
- Docker Compose local development environment
- Deployment to remote server (Docker-based)
