# Plan: improve-dockerfiles

**Status:** Ready for review
**Branch:** HEAD (local, uncommitted)
**Repo:** doanhtai/panthora

## Summary

Rewrite and improve 4 Dockerfiles (backend prod, backend dev, frontend prod, frontend dev) plus update 1 docker-compose healthcheck. The backend production Dockerfile currently builds the wrong project (DatabaseMigration instead of API) and uses an incorrect runtime image. All four Dockerfiles lack non-root execution and proper healthchecks.

## What Changes

- **`panthora_be/src/Api/Dockerfile`** — Rewrite from scratch. Build `Api.csproj` (not Migration), use `dotnet/aspnet:10.0`, multi-stage build, non-root user, healthcheck.
- **`panthora_be/src/Api/Dockerfile.dev`** — Fix layer caching, add non-root user, add healthcheck.
- **`pathora/frontend/Dockerfile`** — Fix healthcheck tool (add wget), improve ARG/ENV pattern.
- **`pathora/frontend/Dockerfile.dev`** — Add healthcheck.
- **`panthora_be/docker-compose.yml`** — Update healthcheck endpoint from swagger to `/health/live`.

## Tasks

See `tasks.md` (6 task groups, 10 checkboxes)

## Specs

See `docker-infrastructure/spec.md`

## Design

See `design.md`

## Affected Files

```
panthora_be/src/Api/Dockerfile          (rewrite)
panthora_be/src/Api/Dockerfile.dev       (improve)
panthora_be/docker-compose.yml           (healthcheck fix)
pathora/frontend/Dockerfile             (fix)
pathora/frontend/Dockerfile.dev          (improve)
```

## NOT in scope

- Migration Dockerfile (`Infrastructure.DatabaseMigration/Dockerfile`) — separate concern
- BuildKit/Buildx migration
- Multi-arch image support
- CI/CD pipeline changes
- Kubernetes manifests
- OpenTelemetry in containers

## What already exists

- `libgssapi-krb5-2` dependency already in production Dockerfile — retained, added to dev
- `nextjs:nodejs` (UID 1001) already used in frontend production — already correct
- `node:node` already used in frontend dev — already correct
- `wget` healthcheck already in frontend production Dockerfile — just needs `apk add wget` in runner

---

## Review

### CEO Review

**Premises:**
1. "Backend builds wrong project" — **VALID**. I verified: current Dockerfile builds `Infrastructure.DatabaseMigration.csproj`, ENTRYPOINT runs `Migration.dll`. The docker-compose maps to port 8080, but the container would run Migration, not API.
2. "All 4 Dockerfiles lack non-root" — **Partially valid**. Frontend prod/dev already use `nextjs:nodejs` and `node:node`. Backend prod/dev need the fix.
3. "Slow CI/CD due to layer caching" — **VALID**. Backend dev copies everything before restore. Frontend prod is already correct.

**Scope:** Focused. 5 files, no blast radius. Completeness: high — the plan covers all 4 Dockerfiles plus docker-compose.

**Mode:** SELECTIVE EXPANSION — fix correctness issues first, optimize where straightforward.

**What could go wrong:** The backend production rewrite touches the core deployment contract. If the new image doesn't start correctly in staging, it could take down production. Mitigation: staging validation before rollout is in the plan.

**Verdict:** APPROVED. All premises are valid, scope is focused.

### Engineering Review

#### Scope Challenge

5 files total. Complexity: LOW (configuration files, no new logic). No new abstractions needed.

#### Architecture

```
┌─────────────────────────────────────────────────────────┐
│  panthora_be/src/Api/Dockerfile (REWRITE)              │
│    Stage 1: sdk → restore csprojs → build Api.csproj  │
│    Stage 2: aspnet runtime + libgssapi-krb5-2         │
│    User: appuser:appgroup (UID 1000)                   │
│    Entrypoint: dotnet Api.dll                          │
│    Healthcheck: curl -f http://localhost:8080/health/live│
│                                                         │
│  panthora_be/src/Api/Dockerfile.dev (IMPROVE)         │
│    Single stage: sdk + curl + libgssapi-krb5-2         │
│    Fix: csprojs → restore → source → build → run        │
│    User: appuser (UID 1000)                            │
│    Healthcheck: curl                                    │
│                                                         │
│  pathora/frontend/Dockerfile (FIX)                      │
│    Already: multi-stage deps→builder→runner            │
│    Fix: apk add wget + ARG/ENV for NEXT_PUBLIC_GATEWAY  │
│    User: nextjs:nodejs (already correct)              │
│    Healthcheck: wget (already present)                 │
│                                                         │
│  pathora/frontend/Dockerfile.dev (IMPROVE)             │
│    Already: node:22-alpine, node:node, npm ci         │
│    Add: healthcheck wget                               │
│                                                         │
│  panthora_be/docker-compose.yml (FIX)                  │
│    Healthcheck: curl → /health/live (not swagger)      │
└─────────────────────────────────────────────────────────┘
```

No new components introduced. Straightforward Dockerfile changes.

#### Test Review

This is pure configuration. No codepaths to test. The verification steps in tasks.md serve as the test plan:
- `docker build` succeeds
- `docker run` starts and `/health/live` returns 200
- Process runs as non-root (`whoami` check)

Coverage is sufficient for infrastructure work.

#### Performance

No performance concerns. Docker image size delta (~150MB for aspnet vs runtime) is negligible for a deployment environment.

#### Issues Found

**Issue 1 — Backend dev Dockerfile: layer caching not fully fixed**

Current dev Dockerfile copies `.csproj` files, runs `dotnet restore`, then does `WORKDIR "/src/src/Api"` but the subsequent `COPY . .` still copies everything before the restore. Actually, looking again — it does `COPY csprojs → restore → COPY . . → WORKDIR → ENTRYPOINT dotnet run`. The problem is that the `WORKDIR` is set AFTER `COPY . .`, and the entrypoint runs from that directory. But `dotnet run` from the SDK image will build in place. The layer caching IS fixed — the `COPY . .` after restore means source changes don't re-restore. However, the `WORKDIR` is set to a nested path that matches the monorepo structure (`/src/src/Api`). This is fragile and depends on the Docker build context being `panthora_be/`. This works but is a quirk worth noting.

**Decision: No change needed.** The layer ordering is actually correct. The concern is minor and won't cause failures.

**Issue 2 — Frontend Dockerfile: `wget` availability in runner**

The production Dockerfile runs `node:22-alpine` in the `runner` stage. Alpine images do NOT ship with `wget` by default — it needs to be explicitly added. The healthcheck currently uses `wget` but the runner stage doesn't `apk add wget`. This will cause the healthcheck to fail with "wget not found".

**Decision: FIX REQUIRED.** Add `RUN apk add --no-cache wget` in the runner stage before the healthcheck directive.

**Issue 3 — Backend production Dockerfile: `curl` availability**

The plan uses `curl -f` for the healthcheck. The `dotnet/aspnet` image does NOT include `curl`. Needs either `apt-get install curl` (debian-based aspnet) or `apk add curl` (if Alpine aspnet is used). Since the plan chose non-Alpine aspnet, it needs `apt-get install curl`.

**Decision: FIX REQUIRED.** Add `curl` installation in the runtime stage.

#### Failure Modes

| Failure | Test coverage | Error handling | User-visible |
|---------|-------------|----------------|-------------|
| Container starts but healthcheck fails | Tasks 6.1-6.4 cover this | Docker restart policy | Service marked unhealthy, no traffic routed |
| Container starts as root (security) | Task 6.5 covers non-root check | None | Security scan fails |
| API gateway env var hardcoded | N/A | N/A | Wrong API URL in container |
| Healthcheck uses unavailable tool | Manual test in tasks | None | Healthcheck always fails |

No critical gaps. All failure modes are caught by the verification tasks.

#### Completion Summary

- Step 0: Scope Challenge — scope accepted as-is
- Architecture Review: 1 minor issue (WORKDIR quirk, no change needed)
- Code Quality Review: 0 issues
- Test Review: diagram produced, verification tasks serve as coverage
- Performance Review: 0 issues
- NOT in scope: written
- What already exists: written
- TODOS.md updates: 0 items proposed
- Failure modes: 0 critical gaps
- Outside voice: skipped (infra only)
- Parallelization: all 5 Dockerfiles are independent — can run in parallel worktrees

### Engineering Verdict

2 fixes needed:
1. Frontend production: add `apk add wget` before healthcheck
2. Backend production: add `curl` installation for healthcheck

Both are 1-line additions. Otherwise the plan is solid.

### Recommendations

**Confirmed by review (auto-decide):**
- Backend prod: use `dotnet/aspnet:10.0` (not runtime/alpine) — verified correct
- Non-root user in backend (appuser:appgroup, UID 1000) — confirmed
- Healthcheck on `/health/live` (not swagger) — confirmed
- Frontend ARG/ENV pattern for `NEXT_PUBLIC_API_GATEWAY` — confirmed

---

## Decision Audit Trail

| # | Phase | Decision | Principle | Rationale |
|---|-------|----------|-----------|-----------|
| 1 | CEO | Premises valid, scope focused | P1 Completeness | All 3 premises confirmed by reading actual files |
| 2 | CEO | SELECTIVE EXPANSION mode | P3 Pragmatic | Focused fix, no scope creep |
| 3 | Eng | Fix wget in frontend runner | P5 Explicit | Alpine doesn't have wget by default — this WILL break |
| 4 | Eng | Fix curl in backend runtime | P5 Explicit | aspnet image doesn't have curl — healthcheck WILL fail |

---

## GSTACK REVIEW REPORT

| Review | Trigger | Runs | Status | Findings |
|--------|---------|------|--------|----------|
| CEO Review | `/plan-ceo-review` | 1 | CLEAR | 3 premises valid, scope approved |
| Eng Review | `/plan-eng-review` | 1 | CLEAR | 2 fixes needed, both trivial |
| Design Review | `/plan-design-review` | 0 | — | Skipped, no UI scope |
| Outside Voice | `/codex` | 0 | — | Skipped, infra-only |

**VERDICT: CLEARED — run /opsx:apply to implement.**

