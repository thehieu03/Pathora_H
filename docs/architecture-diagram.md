# Panthora — System Architecture

<!--
  Style: dark background, white outlined boxes, thin arrows
  Rendering: GitHub (dark mode), Mermaid Live Editor
  For best rendering: theme='dark' in mermaid config
-->

```mermaid
%%{ init: { "theme": "dark", "themeVariables": {
      "background": "#0d1117",
      "primaryColor": "#1f2937",
      "primaryTextColor": "#f0f6fc",
      "primaryBorderColor": "#30363d",
      "lineColor": "#8b949e",
      "secondaryColor": "#21262d",
      "tertiaryColor": "#161b22",
      "nodeBorder": "#30363d",
      "clusterBkg": "#161b22",
      "clusterBorder": "#30363d",
      "edgeLabelBackground": "#0d1117",
      "titleColor": "#f0f6fc"
} } }%%

flowchart TB

    subgraph clients["CLIENTS"]
        direction LR
        browser["🌐 Web Browser"]
        mobile["📱 Mobile"]
    end

    subgraph frontend["FRONTEND  (Next.js 16 + React 18)"]
        direction TB
        nextjs["Next.js App Router"]
        subgraph internals[""]
            direction LR
            rtk["RTK Query\n(server state)"]
            redux["Redux Toolkit\n(global state)"]
            signalr_c["SignalR Client\n(real-time)"]
            i18n["i18next\n(en / vi)"]
            tailwind["Tailwind CSS v4\n(dark mode, RTL)"]
        end
    end

    subgraph backend["BACKEND  (ASP.NET Core 10 — Clean Architecture + CQRS)"]
        direction TB
        kestrel["Kestrel\n:8899 / :7203"]
        subgraph layers[""]
            direction TB
            subgraph api["API Layer"]
                direction LR
                exception_mw["ExceptionHandling\nMiddleware"]
                auth["Auth Middleware\nJWT + Google OAuth"]
                lang_mw["LanguageResolution\nMiddleware"]
                signalr_hub["SignalR Hub\n(WebSocket)"]
                controllers["Controllers"]
            end
            subgraph app["Application Layer — CQRS"]
                direction LR
                commands["Commands\n(Create/Update/Delete)"]
                queries["Queries\n(Get/Search/List)"]
            end
            subgraph domain["Domain Layer"]
                direction LR
                entities["Entities + State Machines\n(Audited, Soft-deleted)"]
                policies["Business Policies\n(Pricing, Deposit, Cancellation)"]
            end
            subgraph infra["Infrastructure Layer"]
                direction LR
                file_mgr["Cloudinary\nFile Manager"]
                outbox["Outbox Worker"]
                sepay["SePay Client"]
            end
        end
    end

    subgraph storage["STORAGE"]
        direction LR
        postgres["PostgreSQL\n34.143.220.132:5432"]
        redis["Redis\n34.143.220.132:6379"]
    end

    subgraph external["EXTERNAL SERVICES"]
        direction LR
        cloudinary["Cloudinary\n(Images / CDN)"]
        google["Google OAuth\n(Login)"]
        sepay_ext["SePay\n(Payment Gateway)"]
        vietqr["VietQR\n(QR Generation)"]
        smtp["SMTP Gmail\n(Transactional Email)"]
        otel["OpenTelemetry\n(Observability)"]
    end

    %% ─── Connections ───────────────────────────────────────────

    clients <-->|"HTTPS / WebSocket"| frontend

    frontend -->|"Axios + Bearer Token"| kestrel
    signalr_c <-->|"WebSocket"| signalr_hub

    kestrel --> exception_mw
    exception_mw --> auth
    auth --> lang_mw
    lang_mw --> controllers
    controllers --> commands
    controllers --> queries

    commands --> domain
    queries --> domain
    domain --> infra

    commands --> outbox
    outbox -.->|"async"| postgres

    commands --> file_mgr
    queries --> redis

    commands --> sepay
    sepay --> sepay_ext

    file_mgr -->|"upload / CDN URL"| cloudinary
    auth --> google
    commands --> vietqr
    commands --> smtp
    outbox --> otel

    commands --> postgres
    queries --> postgres
    commands --> redis
    queries --> redis

    sepay_ext -.->|"webhook"| sepay
    sepay --> signalr_hub
```

---

## Component Summary

### Project: Panthora — B2C/B2B Tour & Travel Platform (Vietnam)

Full lifecycle: tour catalog → booking → payment → operations → post-trip review.

### Frontend (Next.js 16)

| Component | Technology | Purpose |
|-----------|-----------|---------|
| App Router | Next.js 16 + React 18 | SSR, routing |
| State | Redux Toolkit + RTK Query | Global + server state |
| Real-time | SignalR Client | Live data push |
| i18n | i18next (en/vi) | Vietnamese + English |
| Styling | Tailwind CSS v4 | Dark mode, RTL |
| API Gateway | Axios + Bearer token | HTTP client with 401 redirect |
| Auth | Cookie-based (access_token, refresh_token, auth_status) | Cookie: 3000 (prod/Vercel), 3001 (dev) |

### Backend (ASP.NET Core 10)

| Layer | Components |
|-------|-----------|
| **API** | Auth (JWT/Google OAuth), SignalR Hub, Controllers |
| **Application** | CQRS Commands & Queries |
| **Domain** | Entities, State Machines, Business Policies |
| **Infrastructure** | File Manager, Outbox Worker, SePay Client |

### Domain Entities

- **Tour** + **TourClassification** — master product + pricing variants
- **TourInstance** — scheduled departure (Available → Confirmed → SoldOut → InProgress → Completed)
- **Booking** — transaction (Pending → Confirmed → Deposited → Paid → Completed)
- **TourGuide** — guide profiles with ratings, languages
- **User** — customers, admins, guides (JWT + Google OAuth)
- **Review** — post-trip reviews (1-5 stars, approved status)
- **Policies** — Pricing, Deposit, Cancellation, Visa, Insurance
- **Supplier** — hotels, transport, activities providers

### Storage

| Service | Host | Purpose |
|---------|------|---------|
| PostgreSQL | 34.143.220.132:5432 | Primary DB, transactional data |
| Redis | 34.143.220.132:6379 | Session cache, Fusion cache |
| Cloudinary | cloudinary.com | Image upload + CDN (avatars, tour images) |

### External Integrations

| Service | Purpose |
|---------|---------|
| Cloudinary | Image upload, optimization, CDN delivery |
| Google OAuth | Social login |
| SePay | Payment gateway, webhook callbacks |
| VietQR | QR code generation for bank transfers |
| SMTP Gmail | Transactional email |
| OpenTelemetry | Distributed tracing + metrics |

### Ports

| Service | Port |
|---------|------|
| Backend HTTP | 8899 |
| Backend HTTPS | 7203 |
| Frontend | 3000 (prod/Vercel), 3001 (dev) |
| Frontend → Backend | 5182 (default), configurable via `NEXT_PUBLIC_API_GATEWAY` |
| PostgreSQL | 5432 |
| Redis | 6379 (SSL enabled) |

## Data Flows

### Booking Flow
```
User → Frontend → POST /api/booking → Kestrel → Auth → Command
     → Outbox (async) → SignalR Hub → WebSocket → Frontend (live update)
     → SePay → VietQR → User scans QR → Bank transfer
     → SePay webhook → Outbox → Booking status updated → SignalR push
```

### File Upload Flow
```
User → Frontend → TourImageUpload → POST /api/tour (multipart)
     → Kestrel → Auth → Command → File Manager
     → Cloudinary SDK → Cloudinary CDN
     → Return public URL → stored in PostgreSQL
```

### Auth Flow
```
User → Frontend → Login → POST /api/auth/login → JWT (15min access + HttpOnly refresh)
     → Google OAuth → GET /api/auth/google-login → redirect → JWT + cookie
     → Refresh → cookie auto-rotate (168h refresh, 15min access)
```

### Middleware Pipeline
```
Kestrel → ExceptionHandlingMiddleware → CORS → Auth (JWT/Google) → LanguageResolutionMiddleware → Controller
```
