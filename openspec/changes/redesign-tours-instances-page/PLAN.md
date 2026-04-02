# Plan: redesign-tours-instances-page

**Change**: Redesign frontend UI/UX for `/tours?view=instances` page in `pathora/frontend`
**Branch**: (no git remote)
**Status**: APPROVED — ready for implementation

**User approved plan at Final Gate. All review cycles complete. Run `/opsx:apply` to implement.**

---

## Why

The `/tours?view=instances` page currently renders as a generic SaaS listing — dark hero overlay kills aerial photography, filter sidebar looks like a default form, departure cards are cramped with poor visual hierarchy. Users can't quickly scan available departures and prices, hurting conversion. The page must feel like a premium Vietnamese travel brand, not a template.

## What Changes

### Phase 0: Functional Bug Fixes + Backend API (required before redesign)

**Backend changes:**
- `PublicTourInstanceController.cs` — add `classification` and `category` query params
- `GetPublicTourInstancesQuery.cs` — accept `classification` and `category` parameters
- `TourInstanceService.cs` — pass `classification` and `category` to repository
- `TourInstanceRepository.cs` — filter by classification and category in DB query

**Frontend changes:**
- `homeService.getAvailablePublicInstances()` — accept and pass `classification`, `category` params
- `TourDiscoveryPage.tsx` — wire `selectedClassifications`/`selectedCategories` to API call params
- `TourDiscoveryPage.tsx` — add `sortBy` to `useEffect` dependency array
- `FilterDrawer.tsx` — "Show Results" button must trigger re-fetch (not just close drawer)

**Frontend edge fixes (low-risk bug fixes, part of Phase 0):**
- `TourInstanceCard.tsx` — hide "Reserve Now" CTA on soldout/cancelled instances
- `TourDiscoveryPage.tsx` — `reloadKey` state actually triggers re-fetch (add to useEffect deps)

### Phase 1: Visual Redesign
- **Hero Section**: Lighter overlay (warm-tinted, photo-visible), stats bar redesigned as mini icon cards with warm background
- **Search Bar**: Stronger visual presence — increased backdrop opacity, shadow-md, destination chips with warm amber background
- **Filter Sidebar**: Upgraded to premium feel — `--shadow-card` depth (keep existing token), uppercase section labels with letter-spacing, radio-style selectors with `text-teal-600` active state, teal-600 count badge in header
- **Results Toolbar**: Segment-style toggle (filled active state), custom styled sort dropdown with shadow
- **Tour Instance Cards**: 60/40 horizontal layout (desktop), stacked vertical layout (tablet 768-1024px and mobile), 16:9 aspect ratio images, status badge moved to top-right, navy pricing (`text-2xl font-extrabold`), full-width CTA
- **Typography**: Uppercase labels with letter-spacing, navy headings, improved weight contrast, teal-600 for secondary actions
- **Interaction States**: Empty state redesigned (warm illustration + copy + clear filters CTA), loading skeleton redesigned to match new card shape, error state with retry
- **Responsive**: Better desktop density, natural mobile/tablet flow (tablet cards stack vertical)

---

## Capabilities

### New Capabilities

- `tours-instances-ui`: Complete frontend redesign of the tour departure listing page, covering hero, search, filter sidebar, toolbar, and instance cards. Includes functional filter wiring fix.

### Modified Capabilities

<!-- No existing spec-level behavior changes. Filter wiring fix is a bug fix, not a spec change. -->

---

## Affected Files

### Phase 0 — Functional Bug Fixes + Backend
| File | Issue |
|------|-------|
| Backend: `PublicTourInstanceController.cs` | Add `classification`, `category` params to instances API |
| Backend: `GetPublicTourInstancesQuery.cs` | Accept and validate new params |
| Backend: `TourInstanceService.cs` | Pass params to repository |
| Backend: `TourInstanceRepository.cs` | Filter by classification/category in DB query |
| Frontend: `homeService.ts` | Accept and pass new params to API |
| Frontend: `TourDiscoveryPage.tsx` | Wire filters to API call; fix sort deps; fix reloadKey |
| Frontend: `FilterDrawer.tsx` | "Show Results" triggers re-fetch |
| Frontend: `TourInstanceCard.tsx` | Hide CTA on soldout/cancelled |

### Phase 1 — Visual Redesign
| File | Current Pain Points |
|------|---------------------|
| `HeroSection.tsx` | Deep navy overlay kills photo; stats plain text |
| `SearchBar.tsx` | `bg-white/80` too transparent; `shadow-sm` too weak |
| `FilterSidebar.tsx` | Border-only depth; checkboxes with orange active state |
| `FilterDrawer.tsx` | Same as sidebar, mobile context |
| `TourInstanceCard.tsx` | 50/50 grid, 1:1 image, status badge bottom-left |
| `TourDiscoveryPage.tsx` | Tab active state unclear, sort dropdown unstyled |
| `TourCard.tsx` | Polish for consistency |
| `ToursLoading.tsx` | Skeleton doesn't match new card shape |
| Empty state | Currently text-only, no illustration or warm copy |

## Design Decisions Summary

| # | Decision |
|---|----------|
| D1 | Hero overlay: deep navy → warm cream gradient (`rgba(245,243,237,0.15)` → `rgba(10,30,50,0.55)`) |
| D2 | Stats bar: plain text → 3 mini icon cards on `bg-amber-50` |
| D3 | Search bar: `bg-white/80` + `shadow-sm` → `bg-white/95` + `shadow-md` + amber chips |
| D4 | Filter sidebar: border-only → `--shadow-card` (keep existing token) + `text-teal-600` radio selectors |
| D5 | Toolbar tabs: pill-with-border → filled segment control (`bg-[#1a1a2e]` active) |
| D6 | Instance cards: 50/50 + 1:1 → 60/40 (desktop) + 16:9, status top-right, navy pricing |
| D7 | Tablet layout: cards stack vertical (image top, info bottom) at `md` breakpoint (768-1024px) |
| D8 | Teal token: `text-teal-600` (#0d9488) for all teal elements (radio active, count badge, section labels) |
| D9 | Empty state: warm illustration + headline + "Try broadening" suggestion + clear filters CTA |
| D10 | Loading skeleton: redesigned to match new 60/40 horizontal card shape (desktop) + stacked (tablet/mobile) |

## NOT in Scope

- URL schema changes (filter state to URL persistence)
- Dark mode
- New pages or routes
- Changes to `By Tour` view layout (visual polish OK, structural changes no)
- i18n key changes
- Motion/animation specification (hover transitions, filter crossfade, card entrance)
- Returning user signals (price drop, new departures indicators)
- Map view alternative to card grid
- Full WCAG accessibility audit
- Filter debouncing (rapid filter clicks fire concurrent API calls)

## What Already Exists

| Sub-problem | Existing Code |
|-------------|---------------|
| Hero section rendering | `HeroSection.tsx` — already imports Unsplash image, renders headline + stats |
| Search bar with chips | `SearchBar.tsx` — already has sticky behavior, destination chip rendering |
| Filter logic + state | `TourDiscoveryPage.tsx` — already manages filter state, passes to sidebar/drawer |
| Instance card data fetching | `TourDiscoveryPage.tsx` — uses `homeService.getAvailablePublicInstances()` |
| CSS design tokens | `globals.css` — `--landing-navy`, `--landing-accent`, `--accent`, `--shadow-card` |
| Lucide icons | Already installed, `Star`, `Users`, `MapPin`, `SlidersHorizontal` available |
| Tailwind v4 | Already configured, teal/amber scales available |

---

---

## CEO Review

**Mode:** SELECTIVE EXPANSION

### Step 0A — Premise Challenge

| Premise | Assessment |
|---------|-----------|
| "Generic SaaS listing" is the problem | VALID but incomplete — conflates visual design with information architecture |
| Dark hero overlay kills aerial photography | VALID and specific — `rgba(5,7,60,0.7)` drowns Unsplash Ha Long Bay aerial |
| Poor visual hierarchy hurts conversion | UNSUBSTANTIATED — no data cited (bounce rate, session recordings, A/B test) |
| "Users can't quickly scan departures" | PARTIALLY VALID — 50/50 + 1:1 layout is cramped, but root cause may be information density |
| Filter sidebar "looks like a default form" | VALID symptom, weak diagnosis — redesign correctly targets symptoms |

**Critical discovery (from Codex):** `selectedClassifications` and `selectedCategories` are tracked in local state but **never passed to `getAvailablePublicInstances()`** — filters are visually interactive but functionally inert. This is a real functional bug, not just a visual gap.

### Step 0B — Existing Code Leverage Map

| Sub-problem | Existing Code | Leverage |
|---|---|---|
| Hero rendering | `HeroSection.tsx` | Full leverage — overlay + stats only |
| Search bar behavior | `SearchBar.tsx` | Full leverage — visual polish |
| Filter state management | `TourDiscoveryPage.tsx` | EXCLUDED from scope — but state wiring is broken |
| Instance data fetching | `TourDiscoveryPage.tsx` | Full leverage — API call unchanged |
| CSS tokens | `globals.css` | Full leverage — existing tokens |
| Icons | Lucide React | Full leverage |

### Step 0C — Dream State

```
CURRENT → THIS PLAN → 12-MONTH IDEAL
"Generic listing" → "Polished premium page" → "Personalized, data-driven discovery"
(No conversion data) → (Assumption-based redesign) → (A/B tested, analytics-backed)
(Filters non-functional) → (Filters still non-functional) → (Filters wired + URL persistence)
```

### Step 0E — Temporal Interrogation

| Hour | Expected Outcome |
|------|-----------------|
| H1-3 | Hero, search, filters, cards redesigned — immediate visual improvement |
| H4 | Toolbar toggle + polish pass |
| H5-6 | Build + lint + smoke test |
| Risk | CSS cascade issues (shadow → overflow, backdrop → sticky z-index) not budgeted |

### CEO Dual Voices Findings

**CODEX SAYS (strategy challenge):**
- **CRITICAL: Filters are non-functional** — `selectedClassifications`/`selectedCategories` never passed to API. Sort `useEffect` missing `sortBy` in dependency array. These are real bugs, not visual gaps.
- **CRITICAL: No success metric** — plan doesn't define what "better" means (conversion rate, bounce rate, filter usage?)
- **HIGH: Hero overlay change cascades** — title text color (`text-white`), breadcrumb, and stats positioning all need adjustment. Not a single-D1 change.
- **HIGH: Second color system (teal)** — brand already has amber primary + navy. Adding teal creates maintenance debt without defined role boundaries.
- **HIGH: Tablet breakpoint misspecification** — `md:grid-cols-2` at 768-1024px creates cramped 50/50 cards. Plan doesn't address this real pain point.
- **HIGH: By Tour view will break** — shared components (toolbar, sidebar, search) change for instances view but "By Tour" is not redesigned or QA'd.

**CLAUDE CEO SUBAGENT (strategic independence):**
- **CRITICAL: No conversion diagnosis** — redesign driven by aesthetic judgment, not data. Could be optimizing the wrong surface.
- **CRITICAL: No success metric** — cannot evaluate whether redesign worked post-launch.
- **HIGH: 6-month regret** — no lift, wasted effort; tech debt from shared component changes affecting unredesigned views.
- **HIGH: Alternatives not explored** — trust signals (booking count, reviews), price anchoring, social proof ("3 people booked this hour") may convert better than visual polish.
- **MEDIUM-HIGH: Competitive risk** — no competitive teardown of Klook, Viator, Baolau, Traveloka departure pages.

### CEO Consensus

Both voices agree:
1. The plan solves assumed problems without measurement data
2. **Critical functional bug discovered (non-functional filters) that is explicitly excluded from scope**
3. No success metric defined
4. Scope targets visual design while real behavioral issues exist
5. Scope appropriately sized for the visual work, but the visual work may not be the right work

**CROSS-MODEL TENSION:**
- Claude CEO: Root problem = lack of conversion analytics before redesign
- Codex: Root problem = non-functional filters making the page behavior broken
- Resolution: **Both are correct and additive** — analytics would reveal that filters don't work, confirming Codex's finding. This should change scope.

### Scope Modification — USER DECISION: Option A

User chose **Option A** — Fix filters first, then redesign. Scope updated to include:
- Phase 0: Functional filter-to-API wiring fix + sort dependency fix
- Phase 1: Visual redesign (unchanged from original)

## Engineering Review

### Scope Challenge — Resolved

- 9 files across 2 phases — well-scoped, no reduction needed
- No new infrastructure or custom patterns — pure CSS/JSX
- Scope expanded to include backend API changes (Phase 0 now covers full stack filter fix)

### Architecture Findings

**Critical (resolved via scope expansion):** Filter-to-API wiring is impossible without backend changes — `getAvailablePublicInstances()` does not accept classification/category params. Phase 0 expanded to include backend API changes.

**High (resolved):** `handleClearFilters` in FilterDrawer only closes drawer, never triggers re-fetch — fixed by ensuring "Show Results" fires the fetch effect.

**High (resolved):** `reloadKey` state incremented by retry but not in deps — retry is broken — fixed by adding to useEffect deps.

**High (resolved):** "Reserve Now" CTA shown on soldout/cancelled instances — fixed by guarding CTA behind `canReserve` check.

**Medium (deferred):** Two filter systems diverge (singular URL vs array local state for classifications) — requires architecture refactor, out of scope for this PR. Document as TODO.

**Medium (deferred):** `apiLanguage` not in fetch deps — language change doesn't refetch — deferred.

### Code Quality Findings

**High (resolved):** `<button>` nested inside `<Link>` in `TourInstanceCard` — invalid HTML — move CTA outside the Link wrapper.

**High (accepted):** Shared components (sidebar, drawer, search bar) change for instances view, affecting By Tour view. Plan explicitly scopes this — "visual polish OK, structural changes no." QA must verify By Tour view post-implementation.

**Medium (resolved):** Progress bar lacks urgency coloring — add conditional color: green <60%, amber 60-80%, red >80%.

**Medium (deferred):** Status normalization duplicated in `TourInstanceCard` and `homeService` — consolidate into shared utility. Low-risk, deferred.

**Low (accepted):** URL doesn't persist sort order — accepted, out of scope.

**Low (accepted):** No filter debouncing — rapid clicks fire concurrent requests — deferred.

### Test Coverage

**High (in tasks):** Add tests for Phase 0 filter wiring — `selectedClassifications`/`selectedCategories` trigger re-fetch.

**High (in tasks):** Add test for `sortBy` dependency — sort change triggers re-fetch.

**Medium (in tasks):** Add `TourInstanceCard` edge-case tests — soldout CTA hidden, zero-capacity, null fields.

**Critical (in tasks):** Rewrite or remove `tourDiscoveryPageLayoutContract.test.ts` — raw string matching on CSS classes breaks on every refactor. Convert to behavior tests.

### Performance

No N+1 queries or memory concerns identified. All data fetching is through existing service layer. CSS-only changes have no runtime performance impact.

### Failure Modes

| Failure | Test? | Handler? | Silent? |
|---------|-------|----------|---------|
| Filter API returns 400 on bad sort | No | Backend validation | No |
| Backend doesn't support classification/category params | N/A | N/A | Yes (Phase 0 now covers this) |
| Hero overlay too light — text unreadable | Visual QA | Manual check | Yes |
| Skeleton CLS on slow connection | No | Pre-allocated height | Yes |
| Soldout tour still shows CTA | No (new) | Guard added | No |

**1 critical gap** — hero text contrast on redesigned overlay must be verified manually during QA.

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 1 | issues_open | assumptions-driven, no success metric, non-functional filters discovered |
| Codex Review | `/codex review` | Independent 2nd opinion | 1 | issues_open | 10 strategic blind spots, critical filter bug, scope mismatch |
| Eng Review | `/plan-eng-review` | Architecture & tests (required) | 1 | issues_open | critical filter API gap found, resolved via scope expansion |
| Design Review | `/plan-design-review` | UI/UX gaps | 1 | issues_open | 2/10 states, 3/10 a11y, missing interaction model |

**CROSS-PHASE THEMES:**
- Filter functionality is the most contested area — CEO flagged it as assumption, Codex flagged as broken, Eng confirmed as API gap. Scope expansion (Option C) was the right call.
- Shared component risk (sidebar/toolbar affecting By Tour view) flagged independently by Codex eng, Claude eng, and design review. High-confidence signal — must QA both views.

**VERDICT:** All reviews complete. Scope expanded to full-stack (frontend + backend for filter API). Design decisions made (teal-600, subtle shadow, stacked tablet cards, redesign skeleton, add empty state). Ready for implementation pending user approval at Final Gate.

---

## Design Review

**Initial rating: 4/10** — The plan specifies visual targets for 7 components but omits the interaction model, state specifications, and responsive behavior that would make it implementable.

### Step 0 — Design Scope Assessment

- **DESIGN.md exists?** No. Recommend `/design-consultation` first. Proceeding with universal design principles.
- **UI scope:** 7 components across 3 zones (hero, discovery, cards)
- **Existing patterns:** CSS tokens (`--landing-navy`, `--landing-accent`, `--shadow-card`), Tailwind v4 teal/amber scales, Lucide icons — all already in codebase.

### Step 0.5 — Visual Mockups

Design binary unavailable (no OpenAI API key). Skipped. This is a significant gap — text descriptions cannot substitute for visual mockups for a visual redesign. If the user wants to see what "warm cream gradient + mini stats cards" actually look like, `$D setup` needs an OpenAI key first.

### Dual Voices — Litmus Scorecard

```
DESIGN OUTSIDE VOICES — LITMUS SCORECARD:
═══════════════════════════════════════════════════════════════
  Check                                    Claude  Codex  Consensus
  ─────────────────────────────────────── ─────── ─────── ─────────
  1. Brand unmistakable in first screen?    MARGINAL MARGINAL CONFIRMED
  2. One strong visual anchor?              NO      NO    CONFIRMED
  3. Scannable by headlines only?           NO      NO    CONFIRMED
  4. Each section has one job?            PARTIAL PARTIAL CONFIRMED
  5. Cards actually necessary?              YES     YES   CONFIRMED
  6. Motion improves hierarchy?              UNKNOWN UNKNOWN NOT SPEC'D
  7. Premium without decorative shadows?    NO      NO    CONFIRMED
  ─────────────────────────────────────── ─────── ─────── ─────────
  Hard rejections triggered:                1       1     BOTH: "App UI made of stacked cards"
═══════════════════════════════════════════════════════════════
```

### Pass 1: Information Architecture — 5/10

Hero dominates 50-60% of viewport before a single departure appears. Reading order of full page (hero → search → filters → cards) is implied but not mapped. Filter sidebar and search bar relationship on desktop is ambiguous — are they side-by-side or stacked?

**Fix:** Add spatial hierarchy diagram. Define scroll behavior: compact hero after scroll, or full removal?

### Pass 2: Interaction State Coverage — 2/10

**Every state is unspecified.** Loading, empty, error, partial, filtering-in-progress — zero coverage. The most dangerous omission is the empty state: user applies a filter, gets zero results, and the plan gives no guidance. Emotional arc breaks at the point of most user effort.

**Fix (CRITICAL — add to plan):**

| State | What user sees |
|-------|---------------|
| Loading (initial) | Shimmer skeleton matching redesigned card layout |
| Loading (filter change) | Subtle fade-out/in of results area, filter badge updates |
| Empty (zero results) | Warm illustration + "No departures match your filters" + "Try broadening your search" + clear filters CTA |
| Error (API failure) | Error message with retry button, filters remain active |
| Partial (1-3 results) | Normal grid, no special behavior |
| Filter applied + no results | Same as empty, plus show which filters are active |

### Pass 3: User Journey & Emotional Arc — 4/10

Passive discovery-first arc. No signals for returning users (price drop, new departures). No recovery path when empty state hits. No micro-interactions defined for filter transitions.

**Fix:** Add filter transition micro-interaction: results fade/crossfade when filters change. Add "returning user" consideration (out of scope for this PR but note it).

### Pass 4: AI Slop Risk — 5/10

Plan avoids most slop patterns (no centered 3-column grid, no carousel, no generic icons-in-circles). However: cards ARE the page, and the card redesign uses "shadow-card depth" without defining the shadow value. If `--shadow-card` in globals.css is already subtle, the upgrade may be imperceptible. Also: teal introduced without explicit token definition — developers will pick different shades, producing muddy color inconsistency.

**Fix:** Define exact shadow value target. Define exact teal token (`text-teal-600` or `text-teal-700`).

### Pass 5: Design System Alignment — 4/10

No DESIGN.md exists. Brand tokens (`--landing-navy`, `--landing-accent`) exist but are used inconsistently in the plan (some decisions reference CSS vars, others reference Tailwind utilities). Teal is introduced as "new" but the plan says "leverage existing tokens" — contradiction.

**Fix:** Either all color decisions reference CSS vars OR all reference Tailwind utilities, not both. Add teal as an explicit CSS variable with a defined hex value.

### Pass 6: Responsive & Accessibility — 3/10

"Natural mobile/tablet flow" is not a responsive strategy. Tablet breakpoint (768-1024px) is the worst-specified — current 50/50 card grid is cramped here, plan doesn't resolve it. Accessibility: zero coverage. No contrast checks, touch targets, keyboard nav, ARIA.

**Fix (HIGH):** Specify tablet breakpoint explicitly: what does the card layout become at `md`? 60/40 on 2-col grid or switch to stacked? Add accessibility baseline: all text meets WCAG AA contrast (4.5:1 for body, 3:1 for large text).

### Pass 7: Unresolved Design Decisions

| Decision | If Deferred, What Happens |
|----------|--------------------------|
| Shadow-card target value | Implementer guesses, produces inconsistent depth across components |
| Teal token definition | Developers pick different shades, teal creeps into unintended contexts |
| Loading skeleton design | Implementer ships no skeleton or mismatched skeleton |
| Empty state illustration | Implementer ships "No items found." text, emotional arc breaks |
| Error state design | Implementer ships blank area or generic error box |
| Z-index cascade | Search bar sticky z-index may conflict with hero or toolbar |
| Hero scroll behavior | Implementer guesses, may conflict with LandingHeader sticky |
| Tablet card layout | Current cramped 50/50 stays — known pain point unresolved |
| CTA stack on instance card | "Full-width" is ambiguous for multi-action cards |

### What Already Exists (Design)

- CSS tokens: `--landing-navy`, `--landing-accent`, `--accent`, `--shadow-card` — all should be leveraged, not reinvented
- Existing loading skeleton: `ToursLoading` component — should be redesigned to match new card layout
- Existing empty state: currently text-only — plan must define illustration + copy
- Lucide icons: already in use throughout

### NOT in Scope (Design)

- Motion/animation specification (hover transitions, filter crossfade, card entrance)
- Returning user signals (price drop, new departures)
- Map view alternative to card grid
- Full accessibility audit (WCAG AA compliance verification)
- Design system documentation (DESIGN.md creation)

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 1 | issues_open | assumptions-driven, no success metric, non-functional filters discovered |
| Codex Review | `/codex review` | Independent 2nd opinion | 1 | issues_open | 10 strategic blind spots, critical filter bug, scope mismatch |
| Eng Review | `/plan-eng-review` | Architecture & tests (required) | 0 | — | — |
| Design Review | `/plan-design-review` | UI/UX gaps | 1 | issues_open | 2/10 states, 3/10 a11y, missing interaction model, teal gap, tablet unresolved |

**VERDICT:** CEO + DESIGN reviews have open issues. Eng review required. Scope decision (Option A: fix filters first) already applied. Critical design gaps must be resolved before implementation.
