# Proposal: Fix Tour Management Frontend

## Why

The `tour-management` admin flow in `pathora/frontend` has 4 confirmed defects that degrade the admin editing experience: (1) existing tour thumbnail images don't display when editing, only in the preview step; (2) itinerary activity time fields lack validation for required-ness and logical ordering; (3) the "Add Service" button renders a raw i18n key instead of translated text; and (4) the tour table lacks inline status updates, forcing admins to open the edit modal for every status change. These are surface-level bugs with clear root causes, not architectural work.

## What Changes

- **Bug fix — Thumbnail not displaying in edit mode**: Move `TourImageUpload` outside the Vietnamese-language tab block so it always renders; add `useEffect` to sync `showExistingThumbnail` state with the `existingThumbnail` prop.
- **New validation — Activity start/end times**: Add step-level validation in the itineraries step: `startTime` and `endTime` are required; `endTime` must be strictly greater than `startTime`.
- **Bug fix — Missing i18n key**: Add `tourAdmin.buttons.addService` to both `vi.json` and `en.json` i18n files.
- **New feature — Inline status dropdown**: Add a per-row `<select>` dropdown in the tour management table to change tour status directly, using the existing `tourService.updateTourStatus()` API with loading/disabled states and toast feedback.

## Capabilities

### New Capabilities

- `tour-status-inline-edit`: Inline status update from the tour management table without opening the full edit modal.
- `activity-time-validation`: Validation rules for itinerary activity `startTime`/`endTime` fields in the tour form.

### Modified Capabilities

*(none — all changes are bug fixes or additive features that don't change existing contract behavior)*

## Impact

- **Files modified**: `TourListPage.tsx`, `TourForm.tsx`, `TourImageUpload.tsx`, `vi.json`, `en.json`
- **No API contract changes** — `updateTourStatus` already exists in `tourService`
- **No business logic changes** — all four changes are additive or corrective
- **No breaking changes** — existing functionality is preserved, only corrected or extended
