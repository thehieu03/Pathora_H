## ADDED Requirements

### Requirement: Compact hero with surfaced booking context

The tour instance detail hero section SHALL display key booking decision information (start date, availability, price) within the hero footer area so users can assess booking viability without scrolling. The hero height SHALL NOT exceed 45vh or 480px. The hero SHALL include a gradient fade into the content background below the hero.

#### Scenario: Hero displays on desktop
- **WHEN** user navigates to `/tours/instances/[id]` on a desktop viewport
- **THEN** hero renders at `min-h-[45vh] max-h-[480px]` with a full-bleed background image, gradient overlay, and booking-relevant info (location, duration, spots, start date, base price) visible in the hero footer

#### Scenario: Hero displays on mobile
- **WHEN** user navigates to `/tours/instances/[id]` on a mobile viewport
- **THEN** hero renders at `min-h-[35vh]` with the same content, adjusting typography scale appropriately

### Requirement: Uniform gallery grid

The image gallery SHALL display images in a uniform 2x2 grid where all tiles share the same aspect ratio (16:9 / `aspect-video`). Tiles SHALL NOT have inconsistent border-radius on individual corners. A "View all photos" trigger SHALL be available below the grid to open the lightbox. The lightbox SHALL remain fully functional on all tiles.

#### Scenario: Gallery with 4 or fewer images
- **WHEN** gallery has 1-4 images
- **THEN** tiles render in a 2-column grid with consistent `rounded-2xl` corners, each tile clickable to open lightbox at the correct index

#### Scenario: Gallery with more than 4 images
- **WHEN** gallery has 5 or more images
- **THEN** the first 4 tiles render in the 2x2 grid with a "+N photos" overlay on the last tile; clicking the overlay or a "View all" button opens lightbox

### Requirement: Semantic info card grouping

The info cards below the hero SHALL be organized into 3 semantic sections: Trip Details (location, dates, duration), Capacity (participant count with progress bar), and Pricing (base price highlight). Cards within each group SHALL share visual treatment and spacing. Cards SHALL NOT all appear identical.

#### Scenario: Trip Details section
- **WHEN** the page renders the info sections
- **THEN** Trip Details section displays location, start date, end date, and duration in a compact single-card layout with icon-prefixed rows

#### Scenario: Capacity section
- **WHEN** the page renders the capacity section
- **THEN** the CapacityBar component renders with a labeled progress bar showing current/max participation and spots remaining

#### Scenario: Pricing section
- **WHEN** the page renders the pricing section
- **THEN** the base price per person renders as a highlighted card with the warm amber accent color

### Requirement: Conversion-focused booking card

The sticky booking sidebar card SHALL make the total price the dominant visual element. The CTA button SHALL be full-width and prominently styled. The accent bar at the top of the card SHALL be at least 4px wide. Price breakdown line items SHALL use tabular numbers for alignment. The card SHALL remain sticky on desktop and stack below content on mobile.

#### Scenario: Booking card shows total price
- **WHEN** user adjusts guest counts in the sidebar
- **THEN** the total price updates to reflect `basePrice × totalGuests`, renders at the largest font size in the card, and uses the warm amber accent color

#### Scenario: CTA button styling
- **WHEN** the tour is available (spotsLeft > 0)
- **THEN** CTA renders as a full-width gradient button with warm amber for public tours, blue for private tours, with a shimmer hover effect and a glow box-shadow

#### Scenario: Sold out state
- **WHEN** spotsLeft equals 0
- **THEN** CTA renders as a disabled button with muted styling and "Sold Out" label, guest selector remains functional

### Requirement: i18n for all visible UI strings

All user-facing strings in the booking card, info sections, and status areas SHALL use i18n translation keys. Hardcoded English strings such as "occupied", "spots left", "/person", "days left", guest type labels, and help card text SHALL NOT appear. All translation keys SHALL have both Vietnamese and English values.

#### Scenario: Booking card respects current language
- **WHEN** user has selected Vietnamese language
- **THEN** booking card renders "Người lớn", "Trẻ em", "Trẻ sơ sinh", "Chỗ còn", "Tổng cộng", and all other labels in Vietnamese

- **WHEN** user has selected English language
- **THEN** booking card renders "Adults", "Children", "Infants", "spots left", "Estimated total", and all other labels in English

### Requirement: CSS variable color consistency

All inline hardcoded hex colors on the page SHALL use the appropriate CSS variable token: `#fa8b02` → `var(--landing-accent)`, `#c9873a` → `var(--accent)`, `#05073c` → `var(--tour-heading)`. No hardcoded color values SHALL appear in inline `style={{}}` props within the component.

#### Scenario: Component renders with CSS variables
- **WHEN** the page renders all sections
- **THEN** all color values reference CSS tokens (`--accent`, `--tour-heading`, `--tour-body`, `--tour-caption`, `--tour-divider`, `--tour-surface`, `--tour-surface-raised`, `--tour-surface-muted`) instead of inline hex values
