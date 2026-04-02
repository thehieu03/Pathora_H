## 1. Hero Section Redesign (`HeroSection.tsx`)

- [x] 1.1 Change hero overlay from `rgba(5,7,60,0.7)` to warm cream gradient: `linear-gradient(to bottom, rgba(245,243,237,0.15), rgba(10,30,50,0.55))`
- [x] 1.2 Replace inline stats text with 3 mini icon cards: StarIcon (24 tours), UsersIcon (3,500+), MapPinIcon (50+) on `bg-amber-50 rounded-2xl`
- [x] 1.3 Ensure stats cards use `flex flex-wrap` for mobile wrap behavior

## 2. Search Bar Polish (`SearchBar.tsx`)

- [x] 2.1 Increase backdrop opacity from `bg-white/80` to `bg-white/95`
- [x] 2.2 Increase shadow from `shadow-sm` to `shadow-md`
- [x] 2.3 Update destination chips from plain pills to `bg-amber-50 border-amber-200 hover:bg-amber-100 hover:shadow-sm`
- [x] 2.4 Ensure search bar remains `sticky top-0 z-40`

## 3. Filter Sidebar Upgrade (`FilterSidebar.tsx`)

- [x] 3.1 Remove `border border-slate-100`, replace with `shadow-card` only for depth
- [x] 3.2 Add filter header with Lucide `SlidersHorizontal` icon + "Filters" + teal count badge (`bg-teal-100 text-teal-700 rounded-full w-6 h-6 text-xs flex items-center justify-center`)
- [x] 3.3 Add uppercase section labels: `text-xs font-semibold uppercase tracking-widest text-slate-400` for "Classification" and "Category"
- [x] 3.4 Replace checkbox selectors with radio-style circles: teal (`text-teal-600`) border and fill when active, slate outline when inactive
- [x] 3.5 Add `border-b border-slate-100` dividers between filter sections
- [x] 3.6 Keep active filter pills (black, removable) — no change needed

## 4. Filter Drawer Mobile Polish (`FilterDrawer.tsx`)

- [x] 4.1 Mirror filter header styling from sidebar (icon + label + count badge)
- [x] 4.2 Mirror radio selector styling from sidebar (teal active state)
- [x] 4.3 Mirror section label styling (uppercase, letter-spacing)

## 5. Tour Instance Card Redesign (`TourInstanceCard.tsx`)

- [x] 5.1 Change grid layout from `grid-cols-2` (50/50) to `grid-cols-[3fr_2fr]` (60/40)
- [x] 5.2 Change image aspect ratio from `aspect-square` to `aspect-video` (16:9)
- [x] 5.3 Move status badge from `bottom-3 left-3` to `top-3 right-3 absolute z-10`
- [x] 5.4 Style title: `text-xl font-bold text-[#1a1a2e]` (upgrade from `text-lg font-semibold`)
- [x] 5.5 Style price: `text-2xl font-extrabold text-[#1a1a2e]` (upgrade from `text-xl font-bold text-slate-900`)
- [x] 5.6 Make CTA button full-width of the info column: `w-full justify-center`
- [x] 5.7 Ensure mobile layout stacks vertically: `grid-cols-1` on mobile
- [x] 5.8 Add subtle `border border-slate-100` to card container for depth
- [x] 5.9 Ensure spots progress bar uses `bg-primary` (`#C9873A`)

## 6. Toolbar Toggle Styling (`TourDiscoveryPage.tsx`)

- [x] 6.1 Style active tab: `bg-[#1a1a2e] text-white rounded-full px-5 py-2 text-sm font-semibold`
- [x] 6.2 Style inactive tab: `bg-transparent text-slate-500 hover:text-slate-700 rounded-full px-5 py-2 text-sm font-medium`
- [x] 6.3 Style sort dropdown: add `shadow-card border border-slate-100`, chevron icon

## 7. Visual Polish Pass

- [x] 7.1 Check TourCard (`TourCard.tsx`) for consistency: ensure title `font-bold`, price prominent, hover effects match instance cards
- [x] 7.2 Verify responsive layout at desktop (`xl` breakpoint — 3-col grid), tablet (`md` — 2-col), mobile (1-col)
- [x] 7.3 Check empty state and error state styling still appropriate after parent layout changes
- [x] 7.4 Verify CustomizeBanner CTA styling still works with updated sidebar

## 8. Build & Verify

- [x] 8.1 Run `npm run lint` — must pass with no errors
- [x] 8.2 Run `npm run build` — must build successfully
- [x] 8.3 Manual smoke test: browse `/tours?view=instances` and `/tours` at 3 breakpoints (desktop, tablet, mobile)
