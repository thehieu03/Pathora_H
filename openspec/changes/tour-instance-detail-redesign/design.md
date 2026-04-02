## Context

The tour instance detail page (`TourInstancePublicDetailPage.tsx`, ~1451 lines) is a public-facing page served at `/tours/instances/[id]`. It is a `"use client"` component that fetches data via `homeService.getPublicInstanceDetail()`, renders a full hero + gallery + info cards + booking sidebar layout, and redirects to checkout on CTA click.

Current layout structure: 60vh hero → 2-column content (gallery+cards left, sticky booking card right) with 6 info cards in 2x3 grid. The page uses warm cream background (`--tour-surface-muted: #F7F6F3`), amber accent (`#fa8b02`), deep navy heading color (`--tour-heading: #05073c`), and Outfit font family.

Key constraints:
- Tailwind CSS v4 (CSS-first, no tailwind.config.js for design tokens)
- Existing CSS tokens in `globals.css`: `--tour-*`, `--landing-*`, `--accent`, `--shadow-warm-*`
- No new libraries
- Must preserve all existing functionality (API calls, auth redirect, guest selection, lightbox, scroll reveal)
- Responsive: mobile-first, sidebar becomes bottom-stacked on small screens
- Bilingual: English + Vietnamese i18n

## Goals / Non-Goals

**Goals:**
- Reduce hero height by 25-35% to surface booking info earlier
- Elevate total price and CTA as dominant visual focal points in booking card
- Replace 6 equal info cards with 3 semantic groups
- Standardize gallery to uniform aspect-ratio grid
- Replace hardcoded English strings with i18n keys
- Replace scattered hardcoded hex values with CSS variable tokens
- Tighten typographic hierarchy and spacing rhythm

**Non-Goals:**
- No new API endpoints, no data model changes
- No routing changes
- No state management changes
- No new components outside `TourInstancePublicDetailPage.tsx` (except i18n additions)
- No new external dependencies
- No dark mode support (not currently in scope)

## Decisions

### D1: Hero height reduction — 60vh → 45vh

Reduce hero from `60vh max-h-[600px]` to `min-h-[45vh] max-h-[480px]`. Key info (start date, spots remaining, base price) gets surfaced in the hero footer area instead of buried below. The hero image is still dominant but no longer pushes the entire booking flow below the fold.

**Why not remove the hero entirely?** The hero image is critical for travel editorial feel — it builds emotional connection and trust. Removing it would hurt conversion more than the current height helps.

### D2: Gallery — uniform 2x2 aspect-video grid

Replace the 1-large + 4-thumbnails layout with a `grid grid-cols-2 gap-3` where every cell has `aspect-video` (16:9). All images crop uniformly. The `+N` overlay for extra images moves from individual tile overlay to a proper "view all" trigger.

**Why not a masonry or single large + carousel?** Masonry introduces variable heights that break rhythm. A carousel hides content. The 2x2 grid is scannable, preserves all images at equal prominence, and is trivial to implement.

### D3: Info cards — 3 semantic groups instead of 6 equal cards

Current: 6 `InfoCard` components in 2x3 grid (Location, Start Date, End Date, Duration, Participants, Base Price) — all identical visual weight.

New: Three visually distinct sections:
1. **Trip Details section** (location, dates, duration, deadline) — compact rows in a single card
2. **Capacity section** (participants, progress bar) — unchanged structure but tighter
3. **Pricing highlight** — extracted from the sidebar's breakdown as a preview in the left column

The CapacityBar component stays as-is. The InfoCard component is simplified to a reusable inline layout used within the Trip Details section.

### D4: Booking card — price focal point redesign

Changes within the existing sidebar card:
- **Accent bar**: 4px full-width bar at top instead of 1.5px
- **Total price**: `text-[28px]` (up from 22px), bold weight, warm amber color — this is the single most important number
- **Price breakdown**: smaller font, tabular numbers, cleaner visual separation
- **CTA button**: full-width, `py-4`, stronger `box-shadow` glow, directional shimmer on hover

**Why not move the booking card above the gallery?** On mobile it already sits below content. On desktop, the sticky sidebar is a proven pattern for travel sites. Moving it would require significant layout restructuring with minimal conversion benefit.

### D5: Color token migration

Replace all hardcoded `#fa8b02` with `var(--landing-accent)` and all `#c9873a` with `var(--accent)`. Replace all `#05073c` with `var(--tour-heading)`. This is a find-replace operation that affects ~30 inline `style={{}}` props.

**Why not add new CSS variables?** The tokens `--landing-accent`, `--accent`, `--tour-heading` already exist and are semantically correct. No new variables needed.

### D6: i18n — add missing Vietnamese keys

Add the following keys to both `vi.json` and `en.json` (in the `tourInstance` and `landing.tourDetail` namespaces) to replace hardcoded English strings:

| Key path | Vietnamese value | English value |
|---|---|---|
| `tourInstance.occupied` | "đã đặt" | "occupied" |
| `tourInstance.spotsLeft` | "chỗ còn" | "spots left" |
| `tourInstance.daysLeft` | "ngày còn" | "days left" |
| `tourInstance.perPerson` | "/người" | "/person" |
| `tourInstance.guests` | "khách" | "guests" |
| `tourInstance.totalEstimate` | "Tổng cộng" | "Estimated total" |
| `tourInstance.selectGuests` | "Chọn số khách" | "Select number of guests" |
| `tourInstance.priceMayVary` | "Giá có thể thay đổi theo chiết khấu nhóm" | "Price may vary with group discounts" |
| `landing.tourDetail.adults` | "Người lớn" | "Adults" |
| `landing.tourDetail.children` | "Trẻ em" | "Children" |
| `landing.tourDetail.infants` | "Trẻ sơ sinh" | "Infants" |
| `landing.tourDetail.needHelp` | "Cần hỗ trợ đặt tour?" | "Need help booking?" |
| `landing.tourDetail.hereForYou` | "Chúng tôi luôn sẵn sàng" | "We're here for you" |
| `landing.tourDetail.noPaymentNotice` | "Không yêu cầu thanh toán ngay. Bạn sẽ được chuyển đến trang hoàn tất đặt chỗ." | "No payment required now. You'll be redirected to complete your booking." |

**Why not update ALL strings in the file?** The existing `t("tourInstance.backToTour", "Back to tour")` pattern with English fallbacks is intentional — it provides graceful degradation. We only add keys where there are currently hardcoded English strings with no i18n key.

## Risks / Trade-offs

- **[Risk] Hero text contrast on small images** → Mitigation: The gradient overlay already darkens the hero. Ensure the gradient has enough opacity at the bottom where text sits. Test with the lightest hero images.

- **[Risk] Gallery 2x2 hides more images** → Mitigation: The lightbox is already available on every tile click. Add a "View all N photos" button below the grid that opens the lightbox at index 0.

- **[Risk] i18n keys cause empty strings on fallback** → Mitigation: All keys will have both Vietnamese AND English values added simultaneously. No empty-string fallbacks.

## Migration Plan

1. Edit `TourInstancePublicDetailPage.tsx` — apply all visual changes in a single coherent pass
2. Add missing i18n keys to `vi.json` and `en.json`
3. Run `npm run lint` to catch any JSX/TSX issues
4. Run `npm run build` to verify production build
5. Visual verification via `npm run dev` + browser inspection

No rollback needed — this is a forward-only UI change. The old layout is replaced entirely.

## Open Questions

- Should the "Need Help" card in the sidebar be redesigned too, or kept as-is? (Default: keep minimal redesign, match card style)
- Should the floating social buttons (Facebook, chat) be repositioned or restyled? (Default: keep as-is)
- Should the tabs (Overview/Pricing/Itinerary) content sections get any visual improvements beyond the structural changes? (Default: structural only, no content-level changes)
