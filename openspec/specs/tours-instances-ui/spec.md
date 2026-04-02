# tours-instances-ui

Visual/UX specification for the tour departure listing page (`/tours?view=instances`). All requirements are frontend-only. No business logic, API contracts, or filter/sort/search behavior changes.

## ADDED Requirements

### Requirement: Hero section displays warm-tinted overlay with visible aerial photography

The hero section SHALL display a warm cream-tinted overlay gradient over the Unsplash aerial photography background, transitioning from `rgba(245,243,237,0.15)` at top to `rgba(10,30,50,0.55)` at bottom. The hero headline text SHALL remain readable (contrast ratio ≥ 4.5:1). The gradient SHALL create visual continuity between the hero and the warm cream page background below.

### Requirement: Stats bar renders as mini icon cards on warm amber background

The stats bar SHALL render 3 mini-cards in a horizontal row, each containing: a Lucide icon (Star, Users, MapPin), a large bold number, and a small label text. Each card SHALL use `--accent-muted` (`#FBF3E4`) warm amber background with `rounded-2xl` corners. Cards SHALL NOT overflow on desktop; they SHALL wrap to vertical stack on screens narrower than `sm`.

### Requirement: Search bar has strong visual presence via backdrop and shadow

The sticky search bar SHALL use `bg-white/95` backdrop opacity (up from `bg-white/80`), `border border-slate-200`, and `shadow-md` (up from `shadow-sm`). Destination chips SHALL use `bg-amber-50 border-amber-200` with `hover:bg-amber-100 hover:shadow-sm` transitions. The search bar SHALL remain sticky at top-0 with `z-40`.

### Requirement: Filter sidebar displays as premium panel with teal active states

The desktop filter sidebar SHALL use `shadow-card` (no border) for depth against the warm cream page background. The header SHALL display a Lucide `SlidersHorizontal` icon, "Filters" label, and a teal circle badge showing the count of active filters. Section labels SHALL use `text-xs font-semibold uppercase tracking-widest text-slate-400`. Filter selectors SHALL use radio-style circles with teal (`#0d9488` / `teal-600`) active state. Sections SHALL be separated by `border-b border-slate-100` dividers.

### Requirement: Toolbar view toggle uses filled segment control pattern

The "By Tour / By Departure" toggle SHALL render as a filled segment control. The active segment SHALL use `bg-[#1a1a2e] text-white rounded-full px-5 py-2 text-sm font-semibold`. The inactive segment SHALL use `bg-transparent text-slate-500 hover:text-slate-700 rounded-full px-5 py-2 text-sm font-medium`. The sort dropdown SHALL use `shadow-card` with a chevron icon and `border border-slate-100`.

### Requirement: Tour instance cards use 60/40 horizontal layout with 16:9 images

Tour instance cards SHALL display in a horizontal `grid-cols-[3fr_2fr]` (60/40) layout on desktop. The image column SHALL use `aspect-video` (16:9) landscape ratio with `object-cover`. The status badge SHALL be positioned `absolute top-3 right-3 z-10` (top-right, not bottom-left). The info column SHALL display: title (`text-xl font-bold text-[#1a1a2e]`), dates, duration, classification badge, spots progress bar, price (`text-2xl font-extrabold text-[#1a1a2e]`), and a full-width CTA button (`bg-[#fa8b02] hover:bg-[#e67a00] text-white`).

### Requirement: Instance card status badge renders at top-right of image

The status badge SHALL render as a pill at `absolute top-3 right-3 z-10` of the card image. Badge variants: `confirmed` = green pill (`bg-green-50 text-green-700 border-green-200`), `available` = amber pill (`bg-amber-50 text-amber-700 border-amber-200`), `soldout` = red pill (`bg-red-50 text-red-700 border-red-200`). Each badge SHALL include a checkmark or X icon as appropriate.

### Requirement: All section labels use uppercase letter-spaced styling

All filter section labels, card category labels, and toolbar labels SHALL use `text-xs font-semibold uppercase tracking-widest text-slate-400` styling. This creates consistent label hierarchy across the page.

### Requirement: Responsive layout adapts naturally at mobile and tablet breakpoints

The page SHALL render as single-column stacked layout on mobile (< `lg`). The filter sidebar SHALL be replaced by the `FilterDrawer` on mobile. Instance cards SHALL stack vertically (image top, info bottom) on mobile. Results toolbar SHALL wrap gracefully on narrow screens. Destination chips in search bar SHALL scroll horizontally on mobile.
