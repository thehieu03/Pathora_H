## Why

The `/tours?view=instances` page currently renders as a generic SaaS listing — dark hero overlay kills aerial photography, filter sidebar looks like a default form, departure cards are cramped with poor visual hierarchy. Users can't quickly scan available departures and prices, hurting conversion. The page must feel like a premium Vietnamese travel brand, not a template.

## What Changes

- **Hero Section**: Lighter overlay (warm-tinted, photo-visible), stats bar redesigned as mini icon cards with warm background
- **Search Bar**: Stronger visual presence — increased backdrop opacity, shadow-md, destination chips with warm amber background
- **Filter Sidebar**: Upgraded to premium feel — shadow-card depth, uppercase section labels with letter-spacing, radio-style selectors with teal active state, teal count badge in header
- **Results Toolbar**: Segment-style toggle (filled active state), custom styled sort dropdown with shadow
- **Tour Instance Cards**: 60/40 horizontal layout, 16:9 aspect ratio images (aerial photography strength), status badge moved to top-right, navy pricing (`text-2xl font-extrabold`), full-width CTA
- **Typography**: Uppercase labels with letter-spacing, navy headings, improved weight contrast, teal for secondary actions
- **Responsive**: Better desktop density (reduce loãng), natural mobile/tablet flow

## Capabilities

### New Capabilities

- `tours-instances-ui`: Complete frontend redesign of the tour departure listing page, covering hero, search, filter sidebar, toolbar, and instance cards. No business logic changes — purely visual/UX improvements.

### Modified Capabilities

<!-- No existing spec-level behavior changes. This is a pure frontend/UI redesign. -->

## Impact

**Affected files** (frontend only):

- `src/features/tours/components/TourDiscoveryPage.tsx` — orchestrates the page
- `src/features/tours/components/HeroSection.tsx` — hero redesign
- `src/features/tours/components/SearchBar.tsx` — search bar polish
- `src/features/tours/components/FilterSidebar.tsx` — premium filter sidebar
- `src/features/tours/components/FilterDrawer.tsx` — mobile drawer polish
- `src/features/tours/components/TourInstanceCard.tsx` — instance card redesign
- `src/features/tours/components/TourCard.tsx` — visual polish (optional)
- `src/features/tours/components/TourListHeader.tsx` — currently unused, may be revived
- `src/app/(user)/tours/page.tsx` — no changes needed (dynamic import)
- `src/styles/globals.css` — CSS variables if needed for new colors
- i18n files — same keys, same translations, no changes

**No backend changes. No API contract changes. No filter/sort/search logic changes.**
