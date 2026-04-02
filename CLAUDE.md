# CLAUDE.md

This is a monorepo with two independent projects. Read each project's own `CLAUDE.md` for details.

## Projects

| Project | Stack | Port |
|---------|-------|------|
| `panthora_be/` | ASP.NET Core 10, Clean Architecture + CQRS | 8899 (HTTP), 7203 (HTTPS) |
| `pathora/frontend/` | Next.js 16, React 19, Redux Toolkit, RTK Query | 3000 |

## Commands

**Backend** (`panthora_be/`):
```bash
dotnet build LocalService.slnx && dotnet test LocalService.slnx && dotnet run --project src/Api/Api.csproj
```

**Frontend** (`pathora/frontend/`):
```bash
npm ci && npm run dev    # http://localhost:3000
npm run lint && npm run build
```

## Architecture Summary

**Backend**: Clean Architecture + CQRS. `src/` = app code, `tests/` = xUnit tests. Swagger at `/swagger.json`. Docker Compose for Redis, MinIO, API.

**Frontend**: Next.js App Router. Route groups: `(auth)/` = public, `(dashboard)/` = protected. Middleware uses `auth_status` cookie. RTK Query for API, Redux for global state, AuthContext for auth ops. Axios with bearer token + 401 redirect. i18next (`en`, `vi`). Tailwind CSS v4 (dark mode, RTL, multiple layouts). SignalR for real-time.

**Frontend ↔ Backend**: `NEXT_PUBLIC_API_GATEWAY` (default `http://localhost:5182`). SignalR for live refresh.

## Key Conventions

- `@/*` → `./src/*` (frontend)
- No `any` — prefer `unknown` with type guards
- React Hook Form + Yup for forms
- Services: `src/services/`, endpoints: `src/api/endpoints.ts`
- **No auto-commit/push** — only when requested
- **Pre-commit**: lint + build

## gstack

This project uses [gstack](https://github.com/garrytan/gstack) for all web browsing. Use `/browse` skill from gstack. Never use `mcp__claude-in-chrome__*` tools.

Available skills: `/office-hours`, `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`, `/design-consultation`, `/design-shotgun`, `/design-html`, `/review`, `/ship`, `/land-and-deploy`, `/canary`, `/benchmark`, `/browse`, `/connect-chrome`, `/qa`, `/qa-only`, `/design-review`, `/setup-browser-cookies`, `/setup-deploy`, `/retro`, `/investigate`, `/document-release`, `/codex`, `/cso`, `/autoplan`, `/careful`, `/freeze`, `/guard`, `/unfreeze`, `/gstack-upgrade`, `/learn`.
