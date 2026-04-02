## Context

The `/tours?view=instances` page is the primary discovery surface for Vietnamese tour departures. Built on Next.js 16 App Router + React 19 + Tailwind CSS v4, it orchestrates hero, search, filters, and results via `TourDiscoveryPage.tsx`. Current design suffers from weak visual hierarchy, a dark hero overlay that drowns the Unsplash aerial photography, and generic SaaS styling that undermines the premium travel brand positioning.

**Current pain points:**
- Hero overlay (`rgba(5,7,60,0.7)` deep navy) makes the Ha Long Bay aerial photo nearly invisible
- Stats bar is plain inline text — no visual anchor
- Search bar is too transparent (`bg-white/80`) — feels like an afterthought
- Filter sidebar uses checkboxes with orange active state on a flat white surface — form-like, not premium
- Toolbar tabs use pill-with-border pattern — active state insufficiently distinct
- `TourInstanceCard` uses 50/50 horizontal split with 1:1 image — both image and text are cramped
- Card prices use `slate-900` instead of the brand navy, reducing emphasis
- No section label styling (uppercase + letter-spacing) anywhere

**Constraints:**
- Keep all business logic intact (API calls, filter/sort/search behavior, URL params, i18n)
- No API contract changes
- Stay within existing Tailwind v4 + CSS variable system
- Responsive across mobile (1-col), tablet (2-col), desktop (3-col for tours, 2-col for instances)

## Goals / Non-Goals

**Goals:**
- Hero feels premium and warm — photo visible, stats as mini visual cards
- Search bar has enough presence to anchor the page without competing with hero
- Filter sidebar feels like a curated navigation panel, not a form
- Instance cards prioritize scan speed (date + price visible in one glance) and image impact
- Typography has clear hierarchy: labels, titles, prices, meta text all distinct
- Consistent warm cream + navy + amber palette across all elements

**Non-Goals:**
- No backend or API changes
- No changes to filter/sort/search logic or URL schema
- No dark mode implementation
- No new pages or routes
- No changes to the `By Tour` view (only `view=instances`)

## Decisions

### D1: Hero Overlay — Warm-tinted lighter overlay

**Decision:** Replace `rgba(5,7,60,0.7)` with `linear-gradient(to bottom, rgba(245,243,237,0.15), rgba(10,30,50,0.55))` — warm cream at top fading to lighter slate-navy at bottom.

**Rationale:** The Unsplash Ha Long Bay aerial photo is the page's single strongest visual asset. Deep navy overlay buries it. A warm cream tint at the top (matching the page's `--background` cream) creates visual continuity, while the bottom darkens enough to keep the headline readable. The photo bleeds through more, making the hero feel like a window into Vietnam rather than a dark banner.

**Alternatives considered:**
- Remove overlay entirely → headline legibility suffers on bright photos
- Full light overlay (`rgba(255,255,255,0.3)`) → looks washed out, not premium
- Deep navy kept as-is → current problem, kills the photo

### D2: Stats Bar — Mini icon cards

**Decision:** Replace plain inline stats with 3 mini cards, each with Lucide icon + large number + small label, on `--accent-muted` (`#FBF3E4`) warm amber background.

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  StarIcon  24   │  │  UsersIcon 3.5K │  │  MapPinIcon 50+ │
│    curated      │  │   travelers     │  │  destinations   │
│    tours        │  │    joined       │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

**Rationale:** Social proof stats as mini-cards create a visual anchor in the hero's lower third. Warm amber background connects to the amber CTA. Each card has breathing room — the eye naturally distributes attention across 3 focal points rather than scanning a line of text.

**Alternatives considered:**
- Keep plain text → current state, no visual impact
- Large bold numbers only → loses context (label matters: "curated tours" vs "destinations")

### D3: Search Bar — Stronger presence via backdrop + shadow

**Decision:**
- Backdrop: `bg-white/95` (up from `bg-white/80`)
- Border: `border border-slate-200` (up from `border-slate-200/80`)
- Shadow: `shadow-md` (up from `shadow-sm`)
- Destination chips: `bg-amber-50 border-amber-200` with `hover:bg-amber-100 hover:shadow-sm`

**Rationale:** The search bar sits between hero and content. `bg-white/80` with `shadow-sm` makes it feel like a floating afterthought. `bg-white/95` with `shadow-md` gives it gravitational weight — it anchors the content below it without feeling heavy. Warm amber chips contrast with white background, making destinations feel touchable.

### D4: Filter Sidebar — Shadow-card depth + teal radio selectors

**Decision:**
- Remove `border border-slate-100` — rely entirely on `shadow-card` for depth
- Header: Lucide `SlidersHorizontal` icon + "Filters" label + teal circle badge showing active count
- Section labels: `text-xs font-semibold uppercase tracking-widest text-slate-400`
- Selectors: Radio-style circles (not checkboxes) with teal (`#0d9488`) active state
- Sections separated by subtle `border-b border-slate-100` dividers

**Rationale:** A white surface without a border needs shadow to separate from the warm cream page background. Shadow-card (`0 8px 32px rgba(0,0,0,0.06)...`) provides premium depth. Radio selectors feel more "curated navigation" than checkboxes, which imply tick-off list behavior. Teal (`#0d9488`) as active state provides sophisticated contrast with amber CTA — two accent colors that don't compete. Header count badge gives immediate feedback on active filters.

**Alternatives considered:**
- Keep checkboxes with orange active state → too similar to current state, doesn't feel upgraded
- Add colored sidebar header (navy/teal) → creates a top-heavy look that competes with hero
- Remove sidebar entirely (drawer-only) → desktop users lose quick filter visibility

### D5: Toolbar — Filled segment control for view toggle

**Decision:**
```
Active:  bg-[#1a1a2e] text-white rounded-full px-5 py-2 text-sm font-semibold
Inactive: bg-transparent text-slate-500 hover:text-slate-700 rounded-full px-5 py-2 text-sm font-medium
Sort: Custom styled div with shadow-card, chevron icon, border
```

**Rationale:** Current pill-with-border pattern has insufficient contrast for active state. A filled navy pill for active + ghost for inactive creates instant clarity. The segment control pattern (used by Airbnb, Hopper, Skyscanner) is instantly familiar for travel discovery.

### D6: Tour Instance Card — 60/40 split, 16:9 image, top-right status badge

**Decision:**
```
Desktop: grid-cols-[3fr_2fr] (60/40 split)
Image: aspect-[16/9] object-cover (landscape aerial photography)
Status badge: absolute top-3 right-3 z-10 (top-right, not bottom-left)
Info: p-5, title text-xl font-bold navy, dates text-sm slate-600,
      price text-2xl font-extrabold navy, CTA full-width bg-[#fa8b02]
Mobile: stacked (image top, info bottom), image aspect-video
```

**Rationale:** 16:9 landscape is optimal for the aerial Vietnam photography. 60/40 split gives the image breathing room while keeping the info column comfortable for text. Top-right status badge (`CONFIRMED ✓` in green, `AVAILABLE` in amber, `SOLD OUT` in red) is immediately scannable without being hidden behind the image. Navy pricing at `text-2xl font-extrabold` makes the price the second focal point after the image — critical for conversion.

**Alternatives considered:**
- 50/50 split (current) → both cramped
- Stacked vertical layout → better image but horizontal scan speed lost
- Bottom-left status badge (current) → obscured by content, less visible
- `slate-900` price (current) → doesn't leverage brand navy

### D7: Color System Additions

**Decision:** No new CSS variables needed. Leverage existing tokens:
- Teal: `text-teal-700 bg-teal-50 border-teal-200` (Tailwind defaults match `#0d9488` closely)
- Warm amber bg for stats: `bg-amber-50 border-amber-200`
- Navy for headings/prices: `text-[#1a1a2e]` or `text-landing-navy` (already defined)
- Shadows: use existing `--shadow-card` and `--shadow-card-hover`

**Rationale:** Adding new CSS variables increases maintenance surface. Tailwind's built-in teal and amber scales are close enough to brand colors. The brand-specific values (`--landing-navy`, `--landing-accent`) already exist.

## Risks / Trade-offs

| Risk | Mitigation |
|------|-----------|
| **Layout shift on resize** — changing aspect ratio from 1:1 to 16:9 may cause CLS | Test at 3 breakpoints. Use `aspect-video` (fixed ratio) not `h-[60%]` (dynamic). Accept ~50ms CLS budget hit for image repaint. |
| **Stats mini-cards too wide on mobile** — 3 columns may overflow | Stack vertically on mobile: 3 horizontal items in 1 row on `sm` breakpoint minimum. Or 1 column stacked on `xs`. Use `flex-wrap`. |
| **Filter sidebar shadow on cream bg** — subtle shadow may not separate enough | If `shadow-card` is too light against `--background` (#F7F6F3), add `border border-slate-100/50` back as subtle separator. |
| **Teal + orange accent clash** — two bright colors | Teal is used for secondary (filter active state, badges), orange for primary CTA. They occupy different visual zones — teal left sidebar, orange right content. |
| **TourCard visual regression** — redesign focuses on instances | After implementing instance cards, do a quick visual pass on TourCard to ensure consistency in typography, shadow, and hover effects. |
| **Search bar sticky behavior unchanged** — but with stronger visual weight it may feel heavier when scrolling | Test sticky behavior — if it feels too heavy, reduce shadow to `shadow-sm` and opacity to `bg-white/90`. |

## Migration Plan

**Step 1 — Isolated component work** (each file independent, no breaking changes):
1. `HeroSection.tsx` — overlay gradient, stats mini-cards
2. `SearchBar.tsx` — backdrop opacity, shadow, chip styling
3. `FilterSidebar.tsx` — shadow-card, section labels, radio selectors, header badge
4. `FilterDrawer.tsx` — mirror sidebar changes for mobile consistency
5. `TourInstanceCard.tsx` — aspect ratio, 60/40 layout, status badge position, price styling
6. `TourDiscoveryPage.tsx` — toolbar toggle styling (the parent controls tab layout)

**Step 2 — Visual audit**: Run `npm run lint` + `npm run build` to verify no breakage.

**Step 3 — Smoke test**: Browse the page at `/tours?view=instances` and `/tours` (By Tour view) at 3 breakpoints.

**Rollback**: Revert file changes individually via git. No migration/script needed — all changes are visual/CSS only.

## Open Questions

| # | Question | Decision |
|---|----------|----------|
| OQ1 | Should the `By Tour` view cards also get 60/40 horizontal layout? | **No** — TourCard stays vertical (image-top). It's the overview listing; clicking leads to detail page. Instance cards are action-oriented (reserve), so horizontal scan-speed matters more. |
| OQ2 | Should we extract new sub-components (e.g., `StatsCard`, `FilterSection`)? | **Yes if clean** — HeroSection can extract `StatsCard`, FilterSidebar can extract `FilterSection`. But don't over-engineer — if the JSX fits in one file comfortably, keep it. |
| OQ3 | Should `--tour-dark` (#05073c) be replaced globally? | **No** — it may be used in TourDetail hero which has different context. Only change here is the TourDiscovery hero overlay. |
| OQ4 | i18n needed? | **No** — all text keys remain the same. Only visual presentation changes. |
