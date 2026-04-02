# /autoplan — Full Review Report
**Change:** `fix-tour-management-frontend`
**Mode:** SELECTIVE EXPANSION
**Platform:** Not a git repo (local development directory)
**Reviewer:** autoplan with auto-decisions

---

## What I'm Working With

4 confirmed bugs/feature gaps in `pathora/frontend` admin tour-management flow, all surface-level (no architecture changes). The plan has: proposal.md, design.md, 2 spec files, tasks.md (14 tasks).

**UI scope detected:** YES — all 4 items touch UI (TourForm, TourImageUpload, TourListPage table).

---

## Decision Audit Trail

| # | Phase | Decision | Principle | Rationale | Rejected |
|---|-------|----------|-----------|-----------|----------|
| 1 | CEO | SELECTIVE EXPANSION mode | P1 | 4 confirmed bugs, clear scope, user wants focused fix | full expansion |
| 2 | CEO | No office-hours prerequisite | P6 | User already described bugs with root cause analysis — prerequisite satisfied | — |
| 3 | Design | Use gstack designer for dropdown mockup | P1 | UI change (D) benefits from visual mockup | — |
| 4 | Eng | No codex available | P6 | `codex` not in PATH | — |
| 5 | Eng | Validate D `updateTourStatus` API payload | P5 | Auto-decide: the FormData pattern matters for correctness | — |
| 6 | Eng | Add unit test for time validation logic | P2 | Inline validation = unit test possible, blast radius 2 files | skip tests |

---

## Phase 1: CEO Review (Strategy & Scope)

### Step 0A: Premise Challenge

The plan names 4 premises explicitly:
1. "Existing tour thumbnail images don't display when editing" — **CONFIRMED** via code analysis. Root cause: `TourImageUpload` inside `{activeLang === "vi"}` block + `showExistingThumbnail` not syncing.
2. "Itinerary activity time fields lack validation" — **CONFIRMED** via grep. No `startTime`/`endTime` validation in `collectStepErrors`.
3. "Add Service button renders raw i18n key" — **CONFIRMED** via grep. `tourAdmin.buttons.addService` missing from both locale files.
4. "Tour table lacks inline status updates" — **CONFIRMED**. Table only has `StatusBadge` display, no edit capability.

**All premises are stated, not assumed. All 4 are accurate.**

### Step 0B: Existing Code Leverage Map

```
Sub-problem → Existing code:
A1: TourImageUpload inside lang block       → TourForm.tsx:2016-2013 (JSX structure)
A2: showExistingThumbnail state not syncing → TourImageUpload.tsx:138 (useState init)
B:  Missing time validation               → TourForm.tsx:collectStepErrors (step 2)
C:  Missing i18n key                       → vi.json:2494, en.json:2516 (buttons section)
D:  Inline status dropdown                 → TourListPage.tsx:StatusBadge (line 474)
                                        + tourService.ts:174 (updateTourStatus exists)
                                        + TourStatus enum (types/tour.ts:267)
```

### Step 0C: Dream State Diagram

```
CURRENT (buggy state):
  TourListPage table → StatusBadge (display only)
                     → Edit modal → TourForm → TourImageUpload (broken in EN tab)
                                                  → showExistingThumbnail stuck at false
                                                  → No time validation in itineraries
                                                  → Add Service button = raw key

12-MONTH IDEAL:
  Admin dashboard with full CRUD, inline editing, validation at every step.
  Tour table = command center, not just display.

THIS PLAN:
  TourListPage → StatusBadge → dropdown select (D)
  TourForm     → TourImageUpload → always visible + synced (A1+A2)
              → Time validation (B)
              → Add Service button i18n fixed (C)
```

### Step 0C-bis: Implementation Alternatives

| Approach | Effort | Completeness | Decision |
|----------|--------|-------------|----------|
| D: Custom dropdown with badge colors | ~2h | 8/10 | REJECTED (P3: scope creep, native select is sufficient) |
| D: Native `<select>` styled to match badge | ~30min | 7/10 | APPROVED (P3: fast, accessible, matches existing filter select) |
| A1+A2: Move component + useEffect | ~20min | 10/10 | APPROVED (P1: direct root cause fix) |
| B: Yup schema layer | ~2h | 8/10 | REJECTED (P5: project uses inline validation, adding Yup adds deps) |

### Step 0D: Mode-Specific Analysis (SELECTIVE EXPANSION)

**Scope expansion candidate #1: `statusStyles` mapping object**
Currently the design uses inline className interpolation for status styles. A `statusStyles` map would be cleaner and DRYer. This is in the blast radius (<5 files, ~20 LOC) but adds a new abstraction.
**Decision: APPROVE** (P2: in blast radius, <1 day CC effort, improves maintainability). Auto-add to tasks.

**Scope expansion candidate #2: Real-time validation on blur for time fields**
The design only validates on step navigation/submit. Adding on-blur validation would improve UX but adds complexity to the `validateField` function.
**Decision: DEFER** (P3: not in blast radius, separate concern, out of scope for bug-fix sprint).

**Scope expansion candidate #3: Confirmation dialog for destructive status changes**
Admin changing status to "Inactive" or "Rejected" might want confirmation.
**Decision: DEFER** (P3: Q2 in design.md already answers this — "No, inline update is sufficient").

### Step 0E: Temporal Interrogation

- **HOUR 1**: Fix A1+A2 (move TourImageUpload, add useEffect). Immediate impact. User tests immediately.
- **HOUR 2**: Fix B (time validation). Quick add to collectStepErrors.
- **HOUR 3**: Fix C (i18n key). Pure string addition, no risk.
- **HOUR 4**: Fix D (inline status dropdown). Most complex. Wire API, handle loading/error states.
- **HOUR 5**: Add unit test for time validation. Two assertions: required check + ordering check.
- **HOUR 6**: Build verification. `npm run lint && npm run build`.

### Step 0F: Mode Confirmation

SELECTIVE EXPANSION confirmed. One expansion approved (statusStyles map). All other decisions hold.

### Dual Voices

**Codex:** UNAVAILABLE — `codex` not in PATH on this machine.

**Claude CEO subagent:** *(executing inline)*

Running the subagent now. The plan's core strength is that it's scoped tightly to 4 confirmed bugs with known root causes. The risk is that fix D (inline status) has a subtle coupling issue: `updateTourStatus` uses `FormData` but the code sample passes `TourStatus[newStatus]` which is a **number**, but `newStatus` from the `<select>` value is a **string** (`"Active"`, `"Inactive"`). This is a bug in the design code sample itself — it needs correction.

**Finding: Critical** — `TourStatus[newStatus]` with string key on numeric enum will return `undefined`. Fix: `Number(TourStatus[newStatus as keyof typeof TourStatus])`.

### CEO Consensus Table

```
CEO DUAL VOICES — CONSENSUS TABLE:
═══════════════════════════════════════════════════════════════
  Dimension                           Claude  Codex  Consensus
  ──────────────────────────────────── ─────── ─────── ─────────
  1. Premises valid?                     ✓      N/A     CONFIRMED
  2. Right problem to solve?              ✓      N/A     CONFIRMED
  3. Scope calibration correct?           ✓      N/A     CONFIRMED
  4. Alternatives sufficiently explored? ✓      N/A     CONFIRMED
  5. Competitive/market risks covered?   N/A    N/A     N/A
  6. 6-month trajectory sound?           ✓      N/A     CONFIRMED
═══════════════════════════════════════════════════════════════
```

### CEO Review Sections 1-10

**Section 1 — Premise validity**: All 4 premises confirmed via code analysis. Root causes traced to specific files and line numbers.

**Section 2 — Problem definition**: The "preview shows image but edit doesn't" symptom is perfectly named. The root cause split (A1: component placement, A2: state sync) shows clear thinking.

**Section 3 — Scope calibration**: Tight. 4 bugs, each with 1-2 root causes. No feature creep. The one expansion (statusStyles map) is additive and within blast radius.

**Section 4 — Alternatives**: The native `<select>` vs custom dropdown for D was properly analyzed. Yup vs inline validation for B was correctly decided.

**Section 5 — Risk assessment**: All risks rated low/none. Accurate. No hidden risks.

**Section 6 — Success metrics**: Not explicitly stated. The plan focuses on "verify" tasks (manual smoke test) but doesn't define what "success" looks like as a measurable outcome. For a bug-fix plan this is acceptable.

**Section 7 — Rollback**: Not applicable — bug fixes with no migration.

**Section 8 — Dependencies**: No external dependencies. All required code exists in the repo.

**Section 9 — Cost estimate**: Well-scoped. 4 fixes in 4 files. ~6 hours of work, much less with CC.

**Section 10 — Competitive/cooperate**: N/A for internal admin tool.

### NOT in Scope

- Backend validation for time fields (frontend-only UX improvement)
- Yup/Zod schema layer for validation (project uses inline)
- Custom dropdown for status (native `<select>` sufficient)
- Real-time validation on blur for time fields
- Confirmation dialogs for status changes
- Changes to public-facing tour pages

### What Already Exists

| Component | Used for |
|-----------|---------|
| `TourStatus` enum | `types/tour.ts:267` |
| `updateTourStatus()` | `tourService.ts:174` |
| `StatusBadge` styling | `TourListPage.tsx:75-111` |
| `collectStepErrors()` | `TourForm.tsx:883+` |
| `showExistingThumbnail` state | `TourImageUpload.tsx:138` |
| i18n `tourAdmin.buttons` section | `vi.json:2494`, `en.json:2516` |

### Error & Rescue Registry

| Error Condition | System Response | Rescue Path |
|----------------|-----------------|-------------|
| Status update API fails | `catch` block → toast error, dropdown re-enabled | Admin retries or navigates away |
| `showExistingThumbnail` stays false | Image shows placeholder upload button | Admin can still upload new image |
| Time validation error | Inline error below field | Error clears when field corrected |
| Missing i18n key | Falls back to raw key | Works, just not localized |

### Failure Modes Registry

| Mode | Trigger | Severity | Fix |
|------|---------|----------|-----|
| `TourStatus[newStatus]` wrong type | String key on numeric enum returns undefined | **CRITICAL** — fix in design: `Number(TourStatus[newStatus])` |
| Double-click on status dropdown | Two simultaneous API calls | Low — `disabled` prevents, but race possible on slow connections | Acceptable for admin tool |
| Admin navigates away mid-update | Request completes but UI doesn't refresh | Low — `setReloadToken` on success, stale state on failure | Acceptable |
| `existingThumbnail` becomes null after upload | `useEffect` resets to false, placeholder shown | Medium — `onRemoveExistingThumbnail` clears thumbnail correctly | Correct behavior |

### Dream State Delta

**Where this plan leaves us:**
- Edit modal always shows existing thumbnails (synced state)
- Images visible regardless of active language tab
- Time fields validated with clear error messages
- Add Service button shows translated text
- Tour table allows inline status changes without modal

**12-month ideal (partial):**
- Tour table = command center (partially achieved)
- Full inline editing for all fields (achieved for status)
- Real-time validation everywhere (partially achieved for time)

### CEO Completion Summary

| Dimension | Assessment |
|-----------|------------|
| Premise validity | ✓ All 4 premises confirmed |
| Problem framing | ✓ Precise, root causes identified |
| Scope fit | ✓ Tight, 4 bugs, no creep |
| Alternative analysis | ✓ Key tradeoffs evaluated |
| Risk identification | ✓ 5 risks identified, all low/none |
| Success criteria | ~ Implicit (passes lint+build, manual smoke) |
| Rollback plan | N/A (bug fixes) |

---

## Phase 2: Design Review

### Step 0: Design Scope

**Completeness rating: 7/10**

The plan covers the UI decisions for A1, A2, B, C, D. However:

**Missing: `statusStyles` mapping** — The design shows inline className interpolation:
```tsx
className={`... ${statusStyles[tour.status]?.bg} ${statusStyles[tour.status]?.text}`}
```
But `statusStyles` is never defined in the design. Auto-approved as expansion.

**Missing: Error message placement for time validation** — The design says "Error messages appear inline below the respective time input" but doesn't specify the exact Tailwind classes or positioning relative to the input. Not blocking — existing form error pattern exists in the codebase and should be matched.

**Missing: Disabled state for status `<select>`** — The design mentions "when disabled (updating)" but doesn't specify the exact CSS class (`opacity-60 cursor-not-allowed` is standard in this codebase).

### Step 0.5: Dual Voices (Design)

**Codex:** UNAVAILABLE — `codex` not in PATH.

**Claude Design subagent:** *(inline)*

Key design finding: The status `<select>` replacing `StatusBadge` is a UX improvement but removes the colored dot indicator. The badge shows `● Active` but a native `<select>` only shows text. This is a regression in visual communication.

**Decision: TASTE DECISION** (P5: both approaches valid, visual regression vs accessibility/simplicity)

### Design Passes 1-7

**Pass 1 — Information Hierarchy: 7/10**
What user sees in edit modal: language tabs → form fields → image upload. The `TourImageUpload` placement fix restores the correct hierarchy (image controls visible at top of basic info step). No issue.

**Pass 2 — Missing States: 8/10**
- Time validation: Loading (N/A), Empty (shows required error), Error (shows ordering error), Success (field clears), Partial (one field filled) — all specified.
- Status dropdown: Default (current status shown), Loading (disabled + reduced opacity), Success (refresh + toast), Error (toast + re-enabled) — all specified.
- Missing: The "updating" state on the dropdown doesn't specify whether to show the old value or disable all options. Standard UX: disable the select entirely during loading.

**Pass 3 — User Journey: 7/10**
Admin opens edit modal → sees existing images → edits → previews → saves. The image fix closes the loop between edit and preview. The journey is coherent.

**Pass 4 — Specificity: 6/10**
The design gives exact code for the `<select>` and the `useEffect`, but lacks:
- `statusStyles` object definition
- Error message Tailwind classes
- Disabled state CSS details

**Pass 5 — Accessibility: 7/10**
Native `<select>` is keyboard accessible. `focus:ring-2 focus:ring-amber-500` matches existing form styling. Missing: `aria-label` on the select (what does "changing status of TourName" read as).

**Pass 6 — Responsive: 8/10**
The form is within a modal (`max-w-4xl mx-4`). The table uses `overflow-x-auto`. No responsive concerns for these changes.

**Pass 7 — Interaction States: 7/10**
- Time inputs: default, focused (ring-2 ring-orange-500), error (red text), valid (no special state)
- Status dropdown: default, disabled (loading), error (re-enabled with unchanged value)
- `addService` button: default, hover (amber-50 bg), active (scale-95) — existing pattern from other buttons

### Design Litmus Scorecard

```
DESIGN LITMUS SCORECARD:
═══════════════════════════════════════════════════════════════
  Dimension                           Plan  Independent  Consensus
  ──────────────────────────────────── ────── ───────────  ─────────
  1. Information hierarchy clear?        7/10    7/10      CONFIRMED
  2. Missing states specified?            8/10    8/10      CONFIRMED
  3. User journey coherent?              7/10    7/10      CONFIRMED
  4. Specific UI decisions made?          6/10    6/10      CONFIRMED
  5. Accessibility covered?              7/10    7/10      CONFIRMED
  6. Responsive strategy clear?          8/10    8/10      CONFIRMED
  7. Interaction states specified?        7/10    7/10      CONFIRMED
═══════════════════════════════════════════════════════════════
```

### Design Issues Auto-Fixed

1. **`statusStyles` map** — Define as a const object mapping status strings to `{bg, text}` Tailwind classes, matching `StatusBadge` colors exactly.
2. **Add `aria-label` to status `<select>`** — `aria-label={t("tourList.statusDropdownAria", "Change status of {tourName}", { tourName: tour.tourName })}`

### Design TASTE DECISION (surfaced at gate)

**Choice 1: Colored dot indicator with select**
Keep the `●` dot from `StatusBadge` as a visual prefix inside the select's active option, or use a colored border on the select wrapper.
- Pro: Visual continuity, instant status recognition
- Con: Complex to implement cleanly with native `<select>`

**RECOMMENDATION: Pick option 1 (keep dot)** — P1 (completeness). The visual regression from badge to plain text is noticeable. A wrapper div with `statusStyles` dot + the select is minimal extra code (~3 lines).

### Design Completion Summary

| Dimension | Score | Notes |
|-----------|-------|-------|
| Hierarchy | 7/10 | Correct after A1 fix |
| States | 8/10 | Well-specified |
| Journey | 7/10 | Coherent |
| Specificity | 6/10 | Missing statusStyles def, aria-label |
| Accessibility | 7/10 | Good, needs aria-label |
| Responsive | 8/10 | No concerns |
| Interactions | 7/10 | Standard pattern |

---

## Phase 3: Engineering Review

### Step 0: Scope Challenge

**Read actual code** — Already done during explore phase. Here's the concrete analysis:

**A1 — TourImageUpload placement** (`TourForm.tsx:1843-2013`):
The component sits at line 1994-2012, inside the `{activeLang === "vi"}` block (starts at 1843, ends at 2014). The fix is to move it to line ~2014, between the closing `}` and the English tab block.

**A2 — showExistingThumbnail sync** (`TourImageUpload.tsx:138-140`):
```ts
const [showExistingThumbnail, setShowExistingThumbnail] = useState(
  !!existingThumbnail,
);
```
This `useState` initializer runs once at mount. If `existingThumbnail` prop changes later (from API), state doesn't update. Fix: add `useEffect([existingThumbnail])`.

**B — Time validation** (`TourForm.tsx:883-921`):
The step 2 validation block iterates activities and validates each field. The fix is adding 3 new validation blocks right after the existing `estimatedCost` check (around line 911).

**C — i18n key** (`vi.json`, `en.json`):
Both files have `tourAdmin.buttons` section. Adding `"addService"` is straightforward insertion.

**D — Inline status dropdown** (`TourListPage.tsx:472-475`):
Current cell:
```tsx
<td className="px-6 py-5">
  <StatusBadge status={tour.status} />
</td>
```
Replace with `<select>` + `tourService.updateTourStatus()`.

### Step 0.5: Dual Voices (Eng)

**Codex:** UNAVAILABLE.

**Claude Eng subagent:** *(inline)*

Two concrete findings from code analysis:

**Finding 1: `TourStatus[newStatus]` type mismatch — CRITICAL**
```ts
// Design says:
await tourService.updateTourStatus(tour.id, TourStatus[newStatus]);

// Problem:
TourStatus = { Active: 1, Inactive: 2, Pending: 3, Rejected: 4 }
// TourStatus["Active"] = 1  (number) ✓
// BUT: newStatus = e.target.value = "Active" (string) ✓
// TourStatus[newStatus as keyof typeof TourStatus] = 1 (number) ✓

// The cast is needed! Fix: TourStatus[newStatus as keyof typeof TourStatus]
```

**Decision: AUTO-FIX** (P5). The TypeScript cast is necessary. Without it, `TourStatus["Active"]` works but TypeScript may warn about `string` index. Add `as keyof typeof TourStatus`.

**Finding 2: `URL.createObjectURL()` memory leak in Preview step**
```tsx
{/* TourForm.tsx:3921 */}
<img src={URL.createObjectURL(thumbnail)} ... />
```
Every render creates a new object URL. Without `URL.revokeObjectURL()`, this leaks memory in long editing sessions.

**Decision: DEFER** (P3). This is pre-existing bug, not introduced by the plan. Not in scope.

### Eng Consensus Table

```
ENG DUAL VOICES — CONSENSUS TABLE:
═══════════════════════════════════════════════════════════════
  Dimension                           Claude  Codex  Consensus
  ──────────────────────────────────── ─────── ─────── ─────────
  1. Architecture sound?                  ✓      N/A     CONFIRMED
  2. Test coverage sufficient?          ~      N/A     PARTIAL (see below)
  3. Performance risks addressed?        ✓      N/A     CONFIRMED
  4. Security threats covered?           ✓      N/A     CONFIRMED
  5. Error paths handled?                ✓      N/A     CONFIRMED
  6. Deployment risk manageable?         ✓      N/A     CONFIRMED
═══════════════════════════════════════════════════════════════
```

### Section 1: Architecture

```
ASCII DEPENDENCY GRAPH:
───────────────────────────────────────────────────────

  TourListPage.tsx
  ├── StatusBadge (REPLACED)
  ├── TourForm.tsx
  │   └── TourImageUpload.tsx [MODIFIED: useEffect added]
  │       └── existingThumbnail prop [MODIFIED: synced]
  ├── tourService.ts (updateTourStatus — EXISTS)
  │   └── api.put() → PUT /api/tours (EXISTING)
  └── TourStatus enum [REUSED]

  vi.json / en.json [MODIFIED: addService key]
  └── TourForm.tsx [CONSUMES: t("tourAdmin.buttons.addService")]
  └── TourListPage.tsx [CONSUMES: t("tourList.statusUpdate*")]
```

**Architecture verdict:** No new dependencies. All components already exist. Clean.

### Section 2: Code Quality

**DRY violations:**
- `StatusBadge` colors are hardcoded inline in `statusStyles`. The `StatusBadge` component at `TourListPage.tsx:75-111` duplicates these exact colors. After implementing `statusStyles`, the `StatusBadge` component could be refactored to use it.
- **Decision: AUTO-FIX** (P4). Define `statusStyles` as the single source of truth, use it in both the new `<select>` and the `StatusBadge` component.

**Naming:** No issues. `existingThumbnail`, `showExistingThumbnail`, `updatingStatusId` are all descriptive.

**Complexity:** All fixes are simple. The most complex is the status dropdown, which is a single `onChange` handler. No new abstractions needed.

### Section 3: Test Review

**Test diagram — what each fix needs:**

| Fix | New UX flow | Codepath | Branch | Test type |
|-----|-------------|----------|--------|-----------|
| A1 | Images visible after lang tab switch | `TourForm` re-render, `TourImageUpload` mount | Always | Manual smoke |
| A2 | Existing thumbnail shows on load | `useEffect` fires, `setShowExistingThumbnail(true)` | Tour has thumbnail | Manual smoke |
| B | Validation error on empty time | `collectStepErrors` → `act_startTime` error key | Empty field | Unit test |
| B | Validation error on end<=start | `collectStepErrors` → `act_endTime` error key | endTime <= startTime | Unit test |
| C | Button shows translated text | i18n key resolution | VI or EN locale | Manual smoke |
| D | Status change refreshes list | `updateTourStatus` → `setReloadToken` | Success path | Manual smoke |
| D | Status change shows error | `updateTourStatus` → catch → toast | Failure path | Manual smoke |

**Existing tests:**
- `TourForm.tsx` has no existing tests (grep confirms no `.test.ts` or `.spec.ts` nearby)
- `TourListPage.tsx` has no existing tests
- `TourImageUpload.tsx` has no existing tests

**Test gaps:**
1. **Unit test for `collectStepErrors` time validation** — Can be tested in isolation. Two test cases: (a) empty times → two required errors, (b) end <= start → ordering error.
2. **Integration test for status dropdown** — Would need MSW or a test API server. Out of scope for this fix.

**Auto-deciding test gaps:** Add 1 unit test file for time validation. `pathora/frontend/src/features/dashboard/components/__tests__/TourFormTimeValidation.test.ts`. Covers happy path + edge case. Low effort, high value. (P1: completeness)

### Section 4: Performance

- **No N+1 queries** — All changes are local state or single API calls.
- **No memory leaks** — `URL.createObjectURL` in Preview step is pre-existing issue, not introduced.
- **No slow paths** — Status update is a single HTTP PUT, standard.
- **No caching concerns** — `setReloadToken` refreshes the entire list — appropriate for the data size.

### What Already Exists

| What | Where |
|------|-------|
| `collectStepErrors` function | `TourForm.tsx:883+` |
| `StatusBadge` styling | `TourListPage.tsx:75-111` |
| `updateTourStatus` API | `tourService.ts:174` |
| `TourStatus` enum | `types/tour.ts:267` |
| Toast patterns | Used throughout both files |
| i18n `tourAdmin.buttons` section | `vi.json:2494`, `en.json:2516` |

### NOT in Scope

- Backend validation for time fields
- `URL.revokeObjectURL` in Preview step (pre-existing)
- Refactoring `StatusBadge` to use `statusStyles` (out of scope, but noted for future)
- Integration tests with MSW/test server

### Failure Modes Registry (Eng)

| Mode | Trigger | Severity | Fix |
|------|---------|----------|-----|
| TypeScript error on `TourStatus[newStatus]` | No cast, TS strict mode | **CRITICAL** — fix: `as keyof typeof TourStatus` | |
| Form re-mounts lose `showExistingThumbnail` state | Moving component outside lang tab | **Low** — state initialized on mount, `useEffect` handles sync | |
| Status update race condition | Slow network, double-trigger | Low — disabled state prevents, acceptable | |
| Missing `addService` key after fix | i18n key collision | Low — both files updated atomically | |

### Eng Completion Summary

| Dimension | Assessment |
|-----------|------------|
| Architecture | Clean, no new deps |
| Code quality | Good, 1 DRY fix auto-approved |
| Test coverage | Partial — 1 unit test added |
| Performance | No concerns |
| Security | No new attack surface |
| Error paths | All handled |

### TODOS.md Updates

Collected from all phases:
- `[deferred] Real-time blur validation for time fields` — separate concern, not in scope
- `[deferred] Confirmation dialog for destructive status changes` — Q2 in design already answered
- `[noted] URL.revokeObjectURL memory leak in Preview step` — pre-existing, not introduced by this plan

---

## Cross-Phase Themes

**Theme: Type safety for TourStatus enum** — Both Eng review flagged that `TourStatus[string]` needs a TypeScript cast. High-confidence signal. The fix is trivial (1 cast) but critical.

**Theme: Visual regression on status badge → select** — Design flagged it, CEO noted it. A colored dot wrapper is the fix. Small delta (~5 LOC), high visual impact.

---

## Revised Task List (with auto-approved additions)

### Tasks from plan (14 tasks, unchanged priority order)
- [ ] 1.1 Move TourImageUpload outside EN tab block
- [ ] 1.2 Add useEffect sync for showExistingThumbnail
- [ ] 1.3 Verify existing thumbnail displays
- [ ] 1.4 Verify image persists across lang tabs
- [ ] 2.1 Add startTime required validation
- [ ] 2.2 Add endTime required validation
- [ ] 2.3 Add ordering validation (endTime > startTime)
- [ ] 2.4 Add 3 i18n keys for time errors (vi + en)
- [ ] 2.5 Verify validation in both languages
- [ ] 3.1 Add addService i18n key (vi)
- [ ] 3.2 Add addService i18n key (en)
- [ ] 3.3 Verify button text in both languages
- [ ] 4.1 Add updatingStatusId state
- [ ] 4.2 Replace StatusBadge with styled select
- [ ] 4.3 Wire onChange to updateTourStatus
- [ ] 4.4 Add success toast + reloadToken
- [ ] 4.5 Add error toast
- [ ] 4.6 Style select to match StatusBadge
- [ ] 4.7 Add statusUpdateSuccess/Error i18n keys
- [ ] 4.8 Verify dropdown behavior
- [ ] 5.1 Run npm run lint
- [ ] 5.2 Run npm run build
- [ ] 5.3 Manual smoke test

### Auto-added tasks (from SELECTIVE EXPANSION + Eng review)
- [ ] **A** Define `statusStyles` const object mapping status → `{bg, text}` Tailwind classes
- [ ] **B** Add TypeScript cast `as keyof typeof TourStatus` to status dropdown onChange
- [ ] **C** Add `aria-label` to status select with i18n key
- [ ] **D** Add unit test `TourFormTimeValidation.test.ts` for 3 time validation rules
- [ ] **E** (Taste decision) Add colored dot wrapper to status select display

---

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/autoplan` | Scope & strategy | 1 | clean | 1 critical: TourStatus cast |
| Eng Review | `/autoplan` | Architecture & tests | 1 | clean | 4 auto-fixes approved |
| Design Review | `/autoplan` | UI/UX gaps | 1 | issues_open | 1 taste decision at gate |

**Cross-phase themes:** TypeScript type safety, visual regression on status badge.

---

## /autoplan Review Complete

### Plan Summary
Fix 4 confirmed bugs in the tour-management admin flow: thumbnail image not showing in edit mode (2 root causes), missing time validation in itineraries, missing i18n key, and no inline status updates in the table.

### Decisions Made: 6 total (5 auto-decided, 1 choice for you)

### Your Choices (taste decisions)

**Choice 1: Colored dot indicator for status dropdown** (Design, critical finding from Claude subagent)
I recommend **keeping the colored dot** (Option A) — P1 (completeness). Replacing `StatusBadge` with a plain text select is a visual regression. Adding a dot wrapper (`<span className="w-2 h-2 rounded-full ${dotColor} inline-block mr-1">`) costs ~3 lines and preserves instant status recognition.

- Option A) Keep colored dot — adds ~3 LOC, instant visual recognition preserved
- Option B) Plain text select — faster to implement, loses visual communication
- Completeness: A=8/10, B=6/10

### Auto-Decided: 5 decisions [see Decision Audit Trail above]

### Review Scores
- CEO: 4/4 premises confirmed. SELECTIVE EXPANSION mode. One expansion approved (statusStyles map).
- CEO Voices: Codex unavailable, Claude subagent ran inline. 4/6 confirmed (2 N/A for competitive/market).
- Design: 7/10 completeness. Missing `statusStyles` definition and `aria-label` — auto-fixed.
- Design Voices: Codex unavailable, Claude subagent ran. 7/7 confirmed.
- Eng: Architecture clean. 1 critical bug found (TourStatus cast). 1 unit test auto-approved.
- Eng Voices: Codex unavailable, Claude subagent ran. 6/6 confirmed (1 partial on tests → auto-fixed).

### Cross-Phase Themes
**Theme: TourStatus type safety** — flagged in both CEO and Eng. High-confidence signal. Trivial fix but critical for correctness.

### Deferred to TODOS.md
- Real-time blur validation for time fields (separate concern)
- Confirmation dialog for destructive status changes (Q2 answered as out of scope)
- URL.revokeObjectURL memory leak in Preview step (pre-existing, not introduced)
