# Panthora — Architecture Diagram (ASCII)

**Project: B2C/B2B Tour & Travel Platform (Vietnam)**
**Stack: Next.js 16 + React 18 (Frontend) · ASP.NET Core 10 + CQRS (Backend)**

---

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                               CLIENTS                                         │
│                                                                               │
│    ┌──────────────┐                     ┌──────────────┐                     │
│    │ 🌐 Web Browser │                     │ 📱 Mobile    │                     │
│    └──────┬───────┘                     └──────────────┘                     │
│           │                                                                  │
│           │ HTTPS / WebSocket                                                 │
└───────────┼──────────────────────────────────────────────────────────────────┘
            │
            ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                          FRONTEND (Next.js 16 + React 18)                    │
│                                                                               │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │                        Next.js App Router                               │  │
│  │                                                                        │  │
│  │   (auth)/       → Login, Register, Forgot Password                    │  │
│  │   (dashboard)/  → Dashboard, Tours, Bookings, Customers,               │  │
│  │                  Tour Guides, Reviews, Policies, Settings, Invoice      │  │
│  │   (user)/       → Public pages: Home, Tours, Checkout, Profile        │  │
│  │                                                                        │  │
│  │   Middleware    → auth_status cookie → protected route guard         │  │
│  │   i18next       → English / Vietnamese (cookie sync)                  │  │
│  │   Tailwind v4   → Dark mode, RTL, multiple layouts                    │  │
│  └──────┬──────────────┬──────────────┬──────────────┬──────────────────────┘  │
│         │              │              │              │                          │
│         ▼              ▼              ▼              ▼                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │ RTK Query  │  │   Redux    │  │  SignalR   │  │   i18next  │               │
│  │(api fetch) │  │(glb state) │  │  Client    │  │  (en / vi) │               │
│  └────────────┘  └────────────┘  └─────┬──────┘  └────────────┘               │
│                                         │                                     │
│                                         │ Axios + Bearer Token                │
│                                         ▼                                     │
└─────────────────────────────────────────┼─────────────────────────────────────┘
                                          │
                      ┌───────────────────┴───────────────────┐
                      │           BACKEND (ASP.NET Core 10)      │
                      │          Clean Architecture + CQRS       │
                      │                                          │
                      │  ┌──────────────────────────────────┐   │
                      │  │   Kestrel  →  :8899 / :7203     │   │
                      │  └───────────────┬──────────────────┘   │
                      │                  │                      │
                      │  ┌───────────────▼──────────────────┐  │
                      │  │  ExceptionHandling Middleware    │  │
                      │  └───────────────┬──────────────────┘  │
                      │                  │                      │
                      │  ┌───────────────▼──────────────────┐  │
                      │  │     Auth Middleware              │  │
                      │  │  JWT + Google OAuth + Refresh     │  │
                      │  └───────────────┬──────────────────┘  │
                      │                  │                      │
                      │  ┌───────────────▼──────────────────┐  │
                      │  │  LanguageResolution Middleware   │  │
                      │  └───────────────┬──────────────────┘  │
                      │                  │                      │
                      │  ┌───────────────▼──────────────────┐  │
                      │  │      Controllers + SignalR Hub    │  │
                      │  └───────────────┬──────────────────┘  │
                      │                  │                      │
                      │  ┌───────────────▼──────────────────┐  │
                      │  │        CQRS Handlers            │  │
                      │  │                                 │  │
                      │  │  Commands: Create/Update/Delete │  │
                      │  │  Queries: Get/List/Search       │  │
                      │  │  Outbox: async event dispatch   │  │
                      │  └───────────────┬──────────────────┘  │
                      │                  │                      │
  ┌───────────────────┼──────────────────┼──────────────────────┼────────────┐
  │    STORAGE        │                  │                      │            │
  │                   │                  ▼                      ▼            │
  │  ┌────────────────┴───┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
  │  │    PostgreSQL     │  │   Redis   │  │Cloudinary │  │  SePay    │  │
  │  │  34.143.220.132  │  │ 34.143.220 │  │ (Images/  │  │ (Payment  │  │
  │  │      :5432        │  │   .132     │  │   CDN)    │  │  Gateway) │  │
  │  │                  │  │   :6379    │  │           │  │           │  │
  │  │  Tours           │  │            │  │  Avatars  │  │  Webhook  │  │
  │  │  Bookings        │  │  Session   │  │  Tour     │  │  callbacks│  │
  │  │  Tour Instances   │  │  cache     │  │  images   │  │           │  │
  │  │  Tour Guides      │  │            │  │           │  │           │  │
  │  │  Users            │  │  Fusion    │  │           │  │           │  │
  │  │  Policies         │  │  cache     │  │           │  │           │  │
  │  │  Reviews          │  │(SSL, ssl)  │  │           │  │           │  │
  │  │  Payments         │  │            │  │           │  │           │  │
  │  └──────────────────┘  └────────────┘  └────────────┘  └─────┬──────┘  │
  │                                                                  │          │
  │  ┌──────────────────────────────────────────────────────────────┼───────┐  │
  │  │                        EXTERNAL SERVICES                     │       │  │
  │  │                                                             ▼       │  │
  │  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────┐ │  │
  │  │   │ Google OAuth │  │    VietQR    │  │  SMTP Gmail  │  │ Open   │ │  │
  │  │   │   (Login)    │  │(QR Generate)│  │   (Email)    │  │ Telemetry│ │  │
  │  │   └──────────────┘  └──────────────┘  └──────────────┘  └────────┘ │  │
  │  └──────────────────────────────────────────────────────────────────────┘  │
  │                                                                              │
  │  ═══════════════════════════════════════════════════════════════════════   │
  │  DOMAIN ENTITIES                                                             ║
  │  ═══════════════════════════════════════════════════════════════════════   ║
  │                                                                              │
  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                 │
  │  │     Tour       │  │ TourInstance   │  │   Booking      │                 │
  │  │  + Classification│ │ (departures)  │  │ (Pending→Paid) │                 │
  │  │  + TourDay/     │  │ Available→     │  │ + Participants │                 │
  │  │    Activity     │  │ Completed      │  │ + Payments     │                 │
  │  └────────────────┘  └────────────────┘  └────────────────┘                 │
  │                                                                              │
  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                 │
  │  │   TourGuide    │  │     User       │  │   Policies     │                 │
  │  │ (profile/rating│  │ (JWT+Google)   │  │ Pricing/       │                 │
  │  │  /languages)   │  │ (customer/     │  │ Deposit/       │                 │
  │  │                │  │  admin/guide)  │  │ Cancellation   │                 │
  │  └────────────────┘  └────────────────┘  └────────────────┘                 │
  │                                                                              │
  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                 │
  │  │    Review      │  │   Supplier     │  │  TourRequest   │                 │
  │  │ (1-5 stars,   │  │ (hotels/trans/  │  │  (custom tour  │                 │
  │  │  approved)     │  │  activities)   │  │   inquiries)   │                 │
  │  └────────────────┘  └────────────────┘  └────────────────┘                 │
  │                                                                              │
  └──────────────────────────────────────────────────────────────────────────────┘

════════════════════════════════════════════════════════════════════════════════
PORTS & PROTOCOLS
════════════════════════════════════════════════════════════════════════════════

  Backend HTTP    →  8899          Frontend     →  3000 (prod/Vercel), 3001 (dev)
  Backend HTTPS   →  7203          API Gateway  →  5182 (default), via NEXT_PUBLIC_API_GATEWAY
  PostgreSQL      →  5432          Redis        →  6379 (SSL enabled)
  SMTP           →  587 (TLS)

════════════════════════════════════════════════════════════════════════════════
KEY FLOWS
════════════════════════════════════════════════════════════════════════════════

  BOOKING FLOW
  ────────────
  User → Frontend → POST /api/booking → Kestrel → ExceptionMiddleware
       → Auth → LanguageMiddleware → Command
       → Outbox (async) → SignalR Hub → WebSocket → Frontend (live update)
       → SePay → VietQR → User scans QR → Bank transfer
       → SePay webhook → Outbox → Booking status → SignalR push

  FILE UPLOAD FLOW
  ────────────────
  User → TourImageUpload → POST /api/tour (multipart)
       → Kestrel → ExceptionMiddleware → Auth → LanguageMiddleware
       → Command → Cloudinary File Manager
       → Cloudinary SDK → Cloudinary CDN → public URL
       → stored in PostgreSQL

  AUTH FLOW
  ─────────
  Login   → POST /api/auth/login         → JWT (15min) + HttpOnly cookie (168h refresh)
  OAuth   → GET  /api/auth/google-login  → Google redirect → JWT + cookie
  Refresh → cookie auto-rotate           → new JWT (15min access / 168h refresh)

  MIDDLEWARE PIPELINE
  ────────────────────
  Kestrel → ExceptionHandlingMiddleware → CORS → Auth (JWT/Google) → LanguageResolutionMiddleware → Controller
```
