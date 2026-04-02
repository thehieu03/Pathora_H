## 1. Backend Production Dockerfile

- [x] 1.1 Rewrite `panthora_be/src/Api/Dockerfile` — use `dotnet/aspnet:10.0` runtime, build `Api.csproj` (not Migration), add multi-stage build (restore → build → publish → runner), create `appuser:appgroup` (UID 1000), add `libgssapi-krb5-2` dependency, expose port 8080, add healthcheck via curl against `/health/live`

## 2. Backend Dev Dockerfile

- [x] 2.1 Rewrite `panthora_be/src/Api/Dockerfile.dev` — add `libgssapi-krb5-2` via `apk add`, add `curl` for healthcheck, add non-root `appuser` (UID 1000) via `adduser`, fix layer caching (copy csproj files first → restore → then copy source), add healthcheck via curl against `/health/live` with 30s interval

## 3. Frontend Production Dockerfile

- [x] 3.1 Fix `pathora/frontend/Dockerfile` — add `apk add wget` in runner stage before healthcheck, replace hardcoded `ENV NEXT_PUBLIC_API_GATEWAY=http://api:8080` with `ARG NEXT_PUBLIC_API_GATEWAY` + `ENV` pattern (so docker-compose can pass it at build time)

## 4. Frontend Dev Dockerfile

- [x] 4.1 Update `pathora/frontend/Dockerfile.dev` — add healthcheck using wget against port 3001 (interval 30s, timeout 10s, retries 3, start_period 15s), improve layer ordering if needed

## 5. Docker Compose Updates

- [x] 5.1 Update `panthora_be/docker-compose.yml` — change backend healthcheck from `curl -f http://localhost:8080/swagger/index.html` to `curl -f http://localhost:8080/health/live`

## 6. Verification

- [ ] 6.1 Build backend production image: `docker build -f panthora_be/src/Api/Dockerfile -t panthora-api:test .`
- [ ] 6.2 Verify backend health: `docker run --rm panthora-api:test` → `curl http://localhost:8080/health/live` returns 200
- [ ] 6.3 Build frontend production image: `docker build -f pathora/frontend/Dockerfile -t pathora-web:test .`
- [ ] 6.4 Verify frontend health: `docker run --rm -p 3001:3001 pathora-web:test` → `curl http://localhost:3001` returns 200
- [ ] 6.5 Verify non-root execution: `docker run --rm panthora-api:test whoami` returns `appuser`
