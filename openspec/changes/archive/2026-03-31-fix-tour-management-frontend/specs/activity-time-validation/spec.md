# Activity Time Validation — Capability Spec

## Overview

Adds validation rules for the `startTime` and `endTime` fields of activities within the itineraries step of the tour form (`TourForm`, step 2). Validation is inline, triggered on form submission or step navigation.

## Rules

All rules apply per activity within each day plan.

### 1. Required Fields

- `startTime` is required
- `endTime` is required

If either field is empty, the corresponding error message is displayed under the input.

### 2. Logical Ordering

- `endTime` must be strictly greater than `startTime`

If `endTime <= startTime`, the error is shown on the `endTime` field.

### 3. Error Display

- Error messages appear inline below the respective time input
- Error styling matches the existing form error pattern (red text, small font)
- Errors clear automatically when the field is corrected

## Validation Layer

Validation lives in the existing `collectStepErrors` function in `TourForm.tsx`, step 2 block. No new schema library or external validation layer is introduced.

## i18n Keys

| Key | en | vi |
|-----|----|----|
| `tourAdmin.itineraries.startTimeRequired` | "Start time is required" | "Bắt buộc nhập giờ bắt đầu" |
| `tourAdmin.itineraries.endTimeRequired` | "End time is required" | "Bắt buộc nhập giờ kết thúc" |
| `tourAdmin.itineraries.endTimeMustBeAfterStartTime` | "End time must be after start time" | "Giờ kết thúc phải sau giờ bắt đầu" |

## Edge Cases

- Both fields empty: two separate errors shown
- `startTime` filled, `endTime` empty: only `endTimeRequired` shown
- `startTime` empty, `endTime` filled: only `startTimeRequired` shown
- `startTime` and `endTime` equal (e.g. `09:00` = `09:00`): `endTimeMustBeAfterStartTime` shown — the business rule is that activities must have positive duration
- Backend currently accepts empty strings — this is a frontend UX constraint only, backend validation is out of scope
