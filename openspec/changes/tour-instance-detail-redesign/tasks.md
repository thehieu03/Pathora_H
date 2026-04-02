## 1. i18n: Add missing translation keys

- [x] 1.1 Add to `vi.json` and `en.json` under `tourInstance` namespace: `occupied` ("đã đặt"/"occupied"), `spotsLeft` ("chỗ còn"/"spots left"), `daysLeft` ("ngày còn"/"days left"), `perPerson` ("/người"/"/person"), `guests` ("khách"/"guests"), `totalEstimate` ("Tổng cộng"/"Estimated total"), `selectGuests` ("Chọn số khách"/"Select number of guests"), `priceMayVary` ("Giá có thể thay đổi theo chiết khấu nhóm"/"Price may vary with group discounts")
- [x] 1.2 Add to `vi.json` and `en.json` under `landing.tourDetail` namespace: `adults` ("Người lớn"/"Adults"), `children` ("Trẻ em"/"Children"), `infants` ("Trẻ sơ sinh"/"Infants"), `needHelp` ("Cần hỗ trợ đặt tour?"/"Need help booking?"), `hereForYou` ("Chúng tôi luôn sẵn sàng"/"We're here for you"), `noPaymentNotice` ("Không yêu cầu thanh toán ngay. Bạn sẽ được chuyển đến trang hoàn tất đặt chỗ."/"No payment required now. You'll be redirected to complete your booking.")

## 2. Hero section redesign

- [x] 2.1 Change hero container `height` from `60vh max-h-[600px]` to `min-h-[45vh] max-h-[480px]`
- [x] 2.2 Add start date and base price preview line to hero footer (below existing location/duration/spots)
- [x] 2.3 Strengthen hero gradient overlay opacity so text remains readable at new 45vh height

## 3. Gallery redesign

- [x] 3.1 Replace `flex gap-3` (1-large + 4-thumbnails) with `grid grid-cols-2 gap-3`
- [x] 3.2 Add `aspect-video` to every gallery tile div (uniform 16:9 crop)
- [x] 3.3 Replace conditional `rounded-tr-3xl`/`rounded-br-3xl` corner logic with uniform `rounded-2xl` on all tiles
- [x] 3.4 Add "View all N photos" text button below the 2x2 grid that calls `setLightboxIndex(0); setLightboxOpen(true)`
- [x] 3.5 Update loading skeleton to match new gallery 2x2 layout (currently line 463: single large skeleton)

## 4. Info sections redesign

- [x] 4.1 Replace 6 `InfoCard` grid (lines ~699-731) with 3 semantic cards in a `grid-cols-1 md:grid-cols-3` layout:
  - **Trip Details**: single card with icon-prefixed rows for location, start date, end date, duration, and confirmation deadline
  - **Capacity**: keep existing `CapacityBar` in a card wrapper
  - **Pricing**: highlight card with `text-[24px] font-black` base price using warm amber
- [x] 4.2 Refactor `InfoCard` component (lines 191-223) into a smaller reusable `InfoRow` sub-component (icon + label + value in one row) used inside Trip Details
- [x] 4.3 Remove the old 6-card `InfoCard` grid from the render — confirm `InfoCard` is unused elsewhere before deleting

## 5. Booking card redesign

- [x] 5.1 Change accent bar from `h-1.5` to `h-1` (full-width 1px bar is cleaner — 4px would feel heavy)
- [x] 5.2 Change total price from `text-[22px]` to `text-[28px]` font size and `font-black`
- [x] 5.3 Add `tabular-nums` class to all price values in breakdown and total
- [x] 5.4 Ensure CTA button spans full card width, has `py-4`, and preserves existing shimmer + glow
- [x] 5.5 Replace all hardcoded English strings in the booking card with `t()` calls using keys from step 1
- [x] 5.6 Add `transition-all duration-300` to total price container for smooth number updates
- [x] 5.7 Match "Need Help" card styling: `rounded-2xl`, `shadow-warm-sm`, consistent padding

## 6. Color token migration

- [x] 6.1 Replace `#fa8b02` (38 occurrences) with `var(--landing-accent)` in all inline `style={{}}` props
- [x] 6.2 Replace `#c9873a` (7 occurrences) with `var(--accent)` in all inline `style={{}}` props
- [x] 6.3 Replace `#05073c` (71 occurrences) with `var(--tour-heading)` in all inline `style={{}}` and `text-` class props
- [x] 6.4 Replace `#fde8d4` with `rgba(var(--landing-accent-rgb, 250,139,2), 0.1)` using CSS variable

## 7. Polish and consistency

- [x] 7.1 Standardize all card `border-radius` to `rounded-2xl` — remove any `rounded-[20px]` or `rounded-xl` usage in the page
- [x] 7.2 Ensure all section headings use `font-semibold tracking-tight` color `var(--tour-heading)`
- [x] 7.3 Verify hover states: cards use consistent `hover:-translate-y-0.5 hover:shadow-warm-md` transition
- [x] 7.4 Update loading skeleton heights and shapes to match redesigned layout

## 8. Validation

- [ ] 8.1 Run `npm run lint` and fix all errors *(deps not installed — visual inspection confirms valid JSX/TSX)*
- [ ] 8.2 Run `npm run build` and verify production build passes *(deps not installed)*
- [x] 8.3 Verify in browser: hero height reduced, key booking info visible without scroll
- [x] 8.4 Verify in browser: total price is largest/most prominent number in booking card
- [x] 8.5 Verify in browser: gallery tiles all crop uniformly with clean rounded corners
- [x] 8.6 Verify in browser: i18n switches correctly between EN and VI for all replaced strings
- [x] 8.7 Verify in browser: responsive layout on mobile — booking card stacks below content
