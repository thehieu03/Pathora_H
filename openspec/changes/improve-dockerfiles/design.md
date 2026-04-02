## Context

The Pathora monorepo has two services containerized with Docker:
- **Backend**: ASP.NET Core 10 API (`panthora_be/`) — runs on ports 8080/8081
- **Frontend**: Next.js 16 (`pathora/frontend/`) — runs on port 3001

Current Docker setup consists of 4 Dockerfiles (prod + dev for each service) and 2 docker-compose files.

## Goals / Non-Goals

**Goals:**
- Fix correctness issues (backend prod Dockerfile builds wrong project)
- Ensure all containers run as non-root users
- Add healthchecks to all Dockerfiles
- Optimize Docker layer caching for faster CI/CD builds
- Align healthcheck endpoints with actual API endpoints

**Non-Goals:**
- Migrate to Docker BuildKit or Buildx
- Add multi-arch image support (linux/amd64, linux/arm64)
- Change the base images (stay on Alpine for frontend, mcr.microsoft.com for .NET)
- Modify docker-compose service definitions beyond healthcheck and env var injection
- Add tracing/telemetry to containers (OpenTelemetry already configured in app)

## Decisions

### 1. Backend runtime image: `aspnet` over `runtime`

**Decision**: Use `mcr.microsoft.com/dotnet/aspnet:10.0` (not alpine) for the backend runtime stage.

**Rationale**: The current production Dockerfile uses `dotnet/runtime:10.0-alpine` which is the bare .NET runtime — it does not include ASP.NET Core Kestrel server. The `dotnet/aspnet` image adds the ASP.NET Core hosting stack. While Alpine variants exist (`dotnet/aspnet:10.0-alpine`), the non-Alpine variant avoids libc compatibility issues with `libgssapi-krb5-2`.

**Alternative considered**: Alpine-based images (`dotnet/aspnet:10.0-alpine`). Rejected because `libgssapi-krb5-2` has better compatibility on glibc-based images, and the Kerberos dependency is already in the Dockerfile.

### 2. Non-root user: UID 1000 `appuser` / `appgroup`

**Decision**: Create a dedicated `appuser` (UID 1000) in both backend stages, and use the existing `nextjs` (UID 1001) / `nodejs` group for frontend.

**Rationale**: Standard practice for container security. The `dotnet/sdk` image already creates UID 1000 in some variants; explicit creation ensures consistency. Frontend already uses `nextjs:nodejs` (UID 1001) which is correct.

### 3. Healthcheck strategy: `curl` for backend, `wget` for frontend

**Decision**: Use `curl` for backend (`/health/live`), `wget` for frontend (already in image).

**Rationale**: `curl` is available in the ASP.NET Core image via `libcurl` (part of Kestrel's default setup). The `wget` healthcheck in the frontend Dockerfile currently works because Alpine has wget pre-installed in some variants, but explicitly `apk add wget` should be added to the runner stage for safety. Backend dev Dockerfile will add `curl` via `apk add`.

### 4. Layer caching optimization for backend

**Decision**: Copy only the solution config files and csproj files first, then restore, then copy source code.

**Rationale**: Docker layer order matters: `COPY . .` should come AFTER `dotnet restore` so that source code changes don't invalidate the restore cache layer. The current dev Dockerfile copies everything before restore, meaning any source change invalidates the entire build cache.

### 5. Backend prod: build API, not Migration

**Decision**: Rewrite the production Dockerfile to build the `Api.csproj` project, not `Infrastructure.DatabaseMigration.csproj`.

**Rationale**: The current Dockerfile references Migration in all stages — it builds Migration, publishes Migration, and the final ENTRYPOINT runs Migration DLL. The container should run the API. The Migration project is a separate concern (run as a init container or job, not the main API service).

### 6. Expose port 8080 consistently

**Decision**: Backend exposes only 8080 in production Dockerfile (8081 is gRPC, optional for now).

**Rationale**: docker-compose exposes 8080:8080. The dev Dockerfile already binds `--urls http://0.0.0.0:8080`. Align production to match. 8081 (gRPC) can be added later via `ASPNETCORE_HTTP2_ENABLED` env var.

## Risks / Trade-offs

[Risk] Switching from `dotnet/runtime` to `dotnet/aspnet` slightly increases image size (~150MB).
→ Mitigation: The image size difference is negligible for a server deployment. The functional correctness is more important.

[Risk] Non-root user may not have permission to write logs or access certain paths.
→ Mitigation: ASP.NET Core writes logs to stdout/stderr by default (Serilog configured to console). Any file-based logging should use a volume mount.

[Risk] `curl` availability in ASP.NET Core image — not guaranteed in all variants.
→ Mitigation: Use `which curl || apk add curl` in a RUN directive, or use `wget` as fallback (Alpine has wget in busybox). Better yet, use the HTTP health endpoint via `wget` for consistency.

## Migration Plan

1. **Test in staging**: Build new images with `docker build -f panthora_be/src/Api/Dockerfile -t panthora-api:new .`
2. **Validate healthcheck**: `docker run --rm panthora-api:new` → wait → `curl http://localhost:8080/health/live` returns 200
3. **Deploy**: Push new images, update docker-compose to use new image tags
4. **Monitor**: Watch container logs for the first 5 minutes — Serilog console output should appear normally
5. **Rollback**: Keep old image tag (`panthora-api:old`) — `docker tag` old container to rollback if needed

No database migrations needed — this is purely an infrastructure change.

## Open Questions

1. Should the Migration project be containerized as a separate init container, or run as a Kubernetes Job? Currently it shouldn't be in the API Dockerfile at all.
2. Should we add `HEALTHCHECK` to the database migration Dockerfile too?
3. Should the frontend dev Dockerfile add a `NEXT_PUBLIC_API_GATEWAY` ARG default matching production (`http://api:8080`)?
