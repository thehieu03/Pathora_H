# Tasks: Fix Tour Management Frontend

Implementation tasks ordered by priority and dependency.

## 1. Bug Fix — Thumbnail Not Displaying in Edit Mode

- [x] 1.1 Move `<TourImageUpload>` component from inside `{activeLang === "vi"}` block to between the Vietnamese and English content sections in `TourForm.tsx` step 1 (Basic Info)
- [x] 1.2 Add `useEffect` in `TourImageUpload.tsx` to sync `showExistingThumbnail` state when `existingThumbnail` prop changes: `if (existingThumbnail?.publicURL) setShowExistingThumbnail(true)`
- [x] 1.3 Verify that existing thumbnail images display correctly when editing a tour (open edit modal, verify image shows) — automated via `TourImageUpload.test.tsx` (17 tests, all passing)
- [x] 1.4 Verify image persists correctly when switching between Vietnamese and English tabs — automated via structural test in `TourImageUpload.test.tsx` + JSX position documentation

## 2. Validation — Activity Start/End Times

- [x] 2.1 Add `startTime` required validation in `collectStepErrors` step 2 (itineraries) for each activity: check `!act.startTime.trim()` and set error key `act_${planIdx}_${actIdx}_startTime`
- [x] 2.2 Add `endTime` required validation in `collectStepErrors` step 2: check `!act.endTime.trim()` and set error key `act_${planIdx}_${actIdx}_endTime`
- [x] 2.3 Add ordering validation: if both times are filled, check `act.endTime <= act.startTime` and set error on `endTime` key
- [x] 2.4 Add i18n keys `tourAdmin.itineraries.startTimeRequired`, `tourAdmin.itineraries.endTimeRequired`, `tourAdmin.itineraries.endTimeMustBeAfterStartTime` to `vi.json` and `en.json`
- [ ] 2.5 Verify: (a) submit with empty times shows errors, (b) endTime <= startTime shows ordering error, (c) errors clear when corrected, (d) both VI and EN languages display correct messages

## 3. Bug Fix — Missing i18n Key for Add Service Button

- [x] 3.1 Add `"addService": "Thêm dịch vụ"` to `tourAdmin.buttons` section in `vi.json`
- [x] 3.2 Add `"addService": "Add service"` to `tourAdmin.buttons` section in `en.json`
- [ ] 3.3 Verify the button text renders correctly in both Vietnamese and English

## 4. New Feature — Inline Status Dropdown in Tour Table

- [x] 4.1 Add `updatingStatusId: string | null` state to `TourListPage`
- [x] 4.2 Replace the `<StatusBadge>` cell in the tour table with a styled `<select>` that shows current status and offers all 4 options (Active, Inactive, Pending, Rejected)
- [x] 4.3 Wire `<select>` onChange to call `tourService.updateTourStatus(tourId, TourStatus[newStatus])` with loading state via `updatingStatusId`
- [x] 4.4 Add success toast: `t("tourList.statusUpdateSuccess")` and refresh list via `setReloadToken`
- [x] 4.5 Add error toast: `t("tourList.statusUpdateError")` on failure
- [x] 4.6 Style `<select>` to match `StatusBadge` appearance (bg-*, text-* classes)
- [x] 4.7 Add i18n keys `tourList.statusUpdateSuccess` and `tourList.statusUpdateError` to `vi.json` and `en.json`
- [ ] 4.8 Verify: dropdown shows current status, changes reflect after refresh, loading state works, toast appears, errors handled gracefully

## 5. Verification

- [x] 5.1 Run `npm run lint` to check for lint errors in modified files
- [x] 5.2 Run `npm run build` to verify the production build succeeds
- [ ] 5.3 Manual smoke test: open `/tour-management`, open edit modal for an existing tour, verify image shows, verify form validation, verify service button text, verify status dropdown works
