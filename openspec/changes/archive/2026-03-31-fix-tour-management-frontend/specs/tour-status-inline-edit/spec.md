# Tour Status Inline Edit — Capability Spec

## Overview

Allows admins to change a tour's status directly from the tour management table without opening the full edit modal. The tour list page (`TourListPage`) gains a per-row `<select>` dropdown that calls `tourService.updateTourStatus()` and refreshes the list on success.

## Users & Roles

- **Admin** — can update tour status from the management table

## Behavior

### Inline Status Dropdown

- The "Status" column of the tour table renders a `<select>` element instead of a static badge
- The `<select>` displays the current status of the tour as the selected option
- Dropdown options: `Active`, `Inactive`, `Pending`, `Rejected`
- When admin selects a new status:
  1. The dropdown becomes disabled (prevents double-submit)
  2. `tourService.updateTourStatus(tourId, statusNumber)` is called
  3. On success: the table list is refreshed via `setReloadToken`, and a success toast is shown
  4. On failure: an error toast is shown, dropdown is re-enabled

### State

- `updatingStatusId: string | null` — tracks which row is currently being updated
- No per-row state map needed — status updates are atomic and one-at-a-time

### Edge Cases

- If the API call fails, the dropdown reverts to the current (unchanged) value
- If admin navigates away while a status update is in-flight, the request may complete silently — the list will be stale on next visit
- The `StatusBadge` component is no longer rendered in the table cell — replaced entirely by the `<select>`

## UI Details

- `<select>` is styled to visually match `StatusBadge` using the same `bg-*` and `text-*` Tailwind classes per status
- When disabled (updating), cursor changes to `not-allowed` and opacity is reduced
- Toast: success message uses the translation key `tourList.statusUpdateSuccess`; error uses `tourList.statusUpdateError`

## Technical

- Uses existing `tourService.updateTourStatus(tourId, status: number)` — no API change needed
- Status string-to-number mapping via `TourStatus` constants from `src/types/tour.ts`
- No new hooks or state management patterns introduced

## i18n Keys

| Key | en | vi |
|-----|----|----|
| `tourList.statusUpdateSuccess` | "Status updated successfully" | "Cập nhật trạng thái thành công" |
| `tourList.statusUpdateError` | "Failed to update status" | "Cập nhật trạng thái thất bại" |
