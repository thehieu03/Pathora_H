# Design: Fix Tour Management Frontend

## Context

The `pathora/frontend` admin tour-management flow lives in `src/features/dashboard/components/`. The edit flow is a modal rendered by `TourListPage` containing `TourForm`, which uses a wizard-step architecture. Image handling is delegated to `TourImageUpload` (`src/components/ui/`).

Four issues need resolution:
1. `TourImageUpload` renders inside `{activeLang === "vi"}` block, hiding it when switching to English tab
2. `TourImageUpload` uses local `showExistingThumbnail` state that only initializes once — doesn't sync when `existingThumbnail` prop changes
3. `startTime`/`endTime` fields in the itinerary step have no validation
4. The tour table has no inline status control

## Goals / Non-Goals

**Goals:**
- Fix thumbnail image not displaying in edit mode (A1 + A2)
- Add time field validation for itinerary activities (B)
- Fix missing i18n key for "Add Service" button (C)
- Add inline status dropdown in the tour management table (D)

**Non-Goals:**
- No API contract changes — `updateTourStatus` in `tourService` already exists
- No refactoring of the wizard step architecture
- No changes to the public-facing tour detail page
- No changes to `TourDetailPage` (the preview-only view)

## Decisions

### A1 — Move `TourImageUpload` outside the language tab block

**Decision**: Move `<TourImageUpload>` from inside `{activeLang === "vi" && (...)}` to between the closing `}` and `{activeLang === "en" && (...)}`.

**Why**: Images are not language-specific content. The current placement means:
- If user switches to English tab, `TourImageUpload` unmounts entirely
- If they switch back, React re-mounts it fresh (but `showExistingThumbnail` state is gone since component re-created)
- The English tab shows no image controls at all

**Alternative considered**: Keep `TourImageUpload` in both tabs and deduplicate — rejected because it doubles state complexity and is unnecessary.

### A2 — Sync `showExistingThumbnail` state with `existingThumbnail` prop

**Decision**: Add a `useEffect` in `TourImageUpload` that resets `showExistingThumbnail` whenever `existingThumbnail` changes.

```ts
useEffect(() => {
  if (existingThumbnail?.publicURL) {
    setShowExistingThumbnail(true);
  }
}, [existingThumbnail]);
```

**Why**: The current `useState(!!existingThumbnail)` only sets the initial value. When the parent (`TourForm`) passes a new `existingThumbnail` prop (from API data), the local state doesn't update. Preview step bypasses this by checking `existingThumbnail?.publicURL` directly, which is why it shows the image correctly.

**Alternative considered**: Lift `showExistingThumbnail` state to `TourForm` — rejected because it would require threading more props and the fix is simpler at the component level.

### B — Add time validation in `collectStepErrors`

**Decision**: Add validation in step 2 (itineraries) inside `collectStepErrors` for `startTime` and `endTime`.

```ts
// Required check
if (!act.startTime.trim()) {
  newErrors[`act_${planIdx}_${actIdx}_startTime`] = t(
    "tourAdmin.itineraries.startTimeRequired",
    "Start time is required",
  );
}
if (!act.endTime.trim()) {
  newErrors[`act_${planIdx}_${actIdx}_endTime`] = t(
    "tourAdmin.itineraries.endTimeRequired",
    "End time is required",
  );
}
// Logical order
if (act.startTime.trim() && act.endTime.trim()) {
  if (act.endTime <= act.startTime) {
    newErrors[`act_${planIdx}_${actIdx}_endTime`] = t(
      "tourAdmin.itineraries.endTimeMustBeAfterStartTime",
      "End time must be after start time",
    );
  }
}
```

**Why**: Validation lives alongside other field validations in `collectStepErrors`. No new schema layer (Yup) needed — project uses inline validation. Adding new i18n keys for error messages keeps consistency with existing pattern.

**i18n keys to add** (`tourAdmin.itineraries`):
- `startTimeRequired`: "Start time is required" / "Bắt buộc nhập giờ bắt đầu"
- `endTimeRequired`: "End time is required" / "Bắt buộc nhập giờ kết thúc"
- `endTimeMustBeAfterStartTime`: "End time must be after start time" / "Giờ kết thúc phải sau giờ bắt đầu"

### C — Add missing i18n key

**Decision**: Add `"addService": "Add service" / "Thêm dịch vụ"` to `tourAdmin.buttons` in both `vi.json` and `en.json`.

**Why**: The form uses `t("tourAdmin.buttons.addService")` (line 3418), but the key only exists under `tourAdmin.services.addService`. The `buttons` section in both locale files is missing this entry.

### D — Inline status dropdown in tour table

**Decision**: Replace or augment the `<StatusBadge>` cell in the table with a styled `<select>` that calls `tourService.updateTourStatus()` on change.

```tsx
<select
  value={tour.status}
  disabled={updatingStatusId === tour.id}
  onChange={async (e) => {
    const newStatus = e.target.value;
    setUpdatingStatusId(tour.id);
    try {
      await tourService.updateTourStatus(tour.id, TourStatus[newStatus]);
      setReloadToken((v) => v + 1);
      toast.success(`Status updated to ${newStatus}`);
    } catch {
      toast.error("Failed to update status");
    } finally {
      setUpdatingStatusId(null);
    }
  }}
  className={`px-2.5 py-1 rounded-full text-xs font-bold border-0 cursor-pointer focus:outline-none focus:ring-2 focus:ring-amber-500 ${statusStyles[tour.status]?.bg} ${statusStyles[tour.status]?.text}`}
>
  <option value="Active">Active</option>
  <option value="Inactive">Inactive</option>
  <option value="Pending">Pending</option>
  <option value="Rejected">Rejected</option>
</select>
```

**State per row**:
- `updatingStatusId: string | null` — single value since status updates are rare and one-at-a-time

**Why a `<select>` over a custom dropdown**: Matches existing UI pattern (the filter dropdown in the same page uses native `<select>`), simpler to implement, accessible by default.

**Why reuse `TourStatus` constants**: Prevents hardcoded status strings and ensures consistency with the rest of the codebase.

**Alternative considered**: Custom dropdown with status color badges — rejected for scope. Native `<select>` styled to match `StatusBadge` achieves the goal within the sprint.

## Risks / Trade-offs

- **[Low]** A1 restructure of JSX — minor change, no logic touched
- **[Low]** A2 `useEffect` for state sync — could cause double render on mount, but React 18 batched renders make this safe
- **[Low]** B validation — backend currently accepts empty strings for times; this is purely frontend UX improvement
- **[Low]** D status update — single-row locking with `updatingStatusId` is simple; for high-concurrency scenarios a per-row state map would be better, but out of scope for this fix
- **[None]** C i18n fix — pure string addition, no logic

## Open Questions

- **Q1**: Should the status dropdown show all 4 statuses, or exclude the current one? — **A**: Show all, admin may want to cycle (e.g. active→inactive→active). Current `StatusBadge` already handles display.
- **Q2**: Should the status change show a confirmation dialog? — **A**: No, inline update with toast feedback is sufficient for admin workflow.
- **Q3**: Do we need to track which row is updating if multiple admins edit simultaneously? — **A**: Out of scope for this fix. `setReloadToken` will refresh the list on success.
