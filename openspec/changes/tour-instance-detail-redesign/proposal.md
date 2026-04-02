## Why

The tour instance detail page (`/tours/instances/[id]`) is the primary conversion surface for bookings, but it has significant UX/UI problems: hero dominates too much viewport real estate, booking CTAs are buried below the fold, info cards lack semantic grouping, gallery tiles have inconsistent cropping, typography lacks editorial hierarchy, and mixed English/Vietnamese hardcoded strings undermine trust. These issues collectively reduce booking confidence and conversion.

## What Changes

- **Hero**: Reduce height from 60vh/600px to 45vh/480px (~25-35% smaller), surface key booking info (date, availability, price) in the hero footer so users see the most important data without scrolling
- **Gallery**: Replace 1-large + 4-thumbnails asymmetric layout with a uniform 2x2 aspect-video grid — all tiles consistently cropped, no orphan rounded corners
- **Info Cards**: Replace 6 equal cards in a 2x3 grid with 3 semantic groupings (Trip Details, Capacity, Pricing) using consistent card styling
- **Booking Card**: Elevate total price as the dominant visual focal point, strengthen CTA button with wider gradient bar and stronger glow, improve price breakdown typography
- **Typography & Spacing**: Apply editorial typographic hierarchy (uppercase labels, tracking, tighter headings), consistent spacing rhythm, replace hardcoded `#fa8b02`/`#c9873a` colors with CSS variable tokens
- **i18n**: Replace English hardcoded strings (`"occupied"`, `"spots left"`, `"/person"`, etc.) with Vietnamese i18n keys from existing translation files
- **Color consistency**: Replace scattered hardcoded hex values with CSS variable tokens (`--accent`, `--tour-heading`, `--landing-accent`) throughout the page

## Capabilities

### New Capabilities

- `tour-instance-detail-ui`: Redesigned tour instance detail page covering hero, gallery, info groups, booking card, and typography refinements. All existing functionality (API calls, auth flow, checkout redirect, guest selection, lightbox) is preserved.

### Modified Capabilities

- *(none — this is a pure UI redesign; no API contracts, data flows, routing, auth, or i18n architecture change)*

## Impact

- **Frontend**: `pathora/frontend/src/features/tours/components/TourInstancePublicDetailPage.tsx` (main file, ~1451 lines)
- **i18n**: `pathora/frontend/src/i18n/locales/vi.json` and `en.json` — add missing translation keys for hardcoded English strings
- **CSS**: No new tokens needed — uses existing `--tour-*` and `--landing-*` tokens already defined in `globals.css`
- **No backend impact**, **no API changes**, **no routing changes**, **no state management changes**
