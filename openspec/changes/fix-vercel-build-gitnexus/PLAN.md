# Plan: fix-vercel-build-gitnexus

**Change**: Remove `gitnexus` from `pathora/package.json` to fix Vercel production build failure.
**Branch**: (no git remote)
**Status**: In review

---

## Why

Vercel build thất bại vì `gitnexus@1.4.9` nằm trong `dependencies` của `pathora/package.json`. `gitnexus` là native Node.js addon (C++) phụ thuộc vào `tree-sitter`, cần C++20 headers từ V8/Node 24. Vercel build machine (Node 24.14.1 + gcc/Linux) không hỗ trợ C++20 đầy đủ → `node-gyp build` fail với `#error "C++20 or later required."`. Trên máy local (Windows/MSVC), gitnexus có sẵn pre-built binary nên không lỗi.

## What Changes

1. Remove `gitnexus` khỏi `pathora/package.json` (currently in `dependencies`)
2. Thêm `installCommand: "npm install --omit=dev"` vào `pathora/vercel.json` (currently missing this field)

---

---

## CEO Review (HOLD SCOPE)

**Mode:** HOLD SCOPE — 2-file config change, no scope ambiguity.

### Step 0A — Premise Challenge
| Premise | Assessment |
|---------|-----------|
| Vercel build fails due to gitnexus C++ compilation | VALID — confirmed by build log |
| gitnexus unnecessary for production/runtime | VALID — CLI tool for local analysis |
| Local builds succeed without issue | VALID — Windows/MSVC or pre-built binaries |
| Remove approach is best | VALID — zero footprint, simplest |

### Step 0B — Existing Code Leverage
No existing code partially solves this. Dependency configuration issue, not a code problem.

### Step 0C — Dream State
```
CURRENT: Vercel build FAIL → THIS: Remove gitnexus + omit dev → IDEAL: Builds reliably, zero native addons in prod
```

### Step 0C-bis — Implementation Alternatives
| Approach | Decision |
|----------|----------|
| A. Remove gitnexus from dependencies | CHOSEN — cleanest, zero footprint |
| B. devDependencies + --omit=dev | Rejected — unnecessary complexity |
| C. .npmrc with NPM_CONFIG_OMIT=dev | Rejected — affects local dev, extra file |

### NOT in scope
- Fixing gitnexus to compile on Vercel — not worth the effort
- Backend changes (panthora_be) — unrelated
- Pre-built binary for gitnexus — unnecessary

### What already exists
- `pathora/package.json` — needs gitnexus removed from dependencies
- `pathora/vercel.json` — needs `--omit=dev` added to installCommand
- Both files already in correct location

### Error & Rescue Registry
N/A — no code being written.

### Failure Modes Registry
| Codepath | Failure | Rescued? | Test? | User Sees |
|----------|---------|-----------|-------|-----------|
| vercel.json installCommand | Invalid flag syntax | N/A | No | Vercel build fails immediately |
| package.json edit | JSON syntax error | N/A | No | Vercel build fails immediately |

Both failure modes are immediately obvious on Vercel rebuild. Not silent.

### CEO Dual Voices
Both unavailable: Codex (no git repo in trusted path), Claude subagent (empty output).
Single-reviewer mode.

### CEO Completion Summary
```
| Premise challenge       | 4 premises, all valid                |
| Alternatives           | 3 approaches, 1 chosen                |
| NOT in scope           | 3 items deferred                      |
| Dream state            | Written                               |
| Error/rescue registry  | N/A                                   |
| Failure modes          | 2 identified, non-critical            |
| Dual voices            | Unavailable (both failed)              |
```

---

## Phase 1 Complete

CEO: 4 premises validated, 3 alternatives evaluated, HOLD SCOPE confirmed.
CEO Voices: unavailable (Codex + subagent both failed).
Consensus: N/A — single-reviewer mode.
Passing to Phase 2 (skipped, no UI scope) → Phase 3.

---

## Phase 3: Eng Review

### Step 0: Scope Challenge

**Files changed:** 2 (both JSON config)
**New code:** 0
**Complexity:** Trivial

#### Actual code analysis:
- `pathora/package.json` — remove `"gitnexus": "^1.4.9"` from dependencies object
- `pathora/vercel.json` — add `"installCommand": "npm install --omit=dev"` (currently has `buildCommand` and `outputDirectory` only)

#### Search check
N/A — standard npm/Vercel config, no unfamiliar patterns.

#### TODOS cross-reference
No TODOS.md found.

### Architecture ASCII Diagram
```
BEFORE:
  pathora/
    package.json ──── gitnexus@^1.4.9 ── [tree-sitter native] ── node-gyp build
    vercel.json ──── installCommand: "npm install"          ── installs ALL deps
                                                                  └── FAILS on Vercel

AFTER:
  pathora/
    package.json ──── (gitnexus removed)
    vercel.json ──── installCommand: "npm install --omit=dev" ── installs PROD deps only
                                                                  └── SUCCESS
```

### Section 1 — Architecture
No new architecture. No new components. No data flows. No security surface changes. No scaling concerns.

### Section 2 — Code Quality
Reviewed:
- `pathora/package.json` — valid JSON, removing one key from dependencies object
- `pathora/vercel.json` — valid JSON, adding one key to root object

No issues found. Both are standard JSON files.

### Section 3 — Test Review

No new code paths. No testable behavior. The build passing is verifiable by Vercel rebuild.

### Section 4 — Performance
No performance impact. npm install skips fewer packages on Vercel (slightly faster install). Marginal improvement.

### Eng Dual Voices
Both unavailable: Codex (no git repo), Claude subagent (empty output).
Single-reviewer mode.

### NOT in scope (Eng)
- Any other dependency changes
- Backend (panthora_be)
- gitnexus binary for local dev

### What already exists (Eng)
- `pathora/frontend/vercel.json` — separate deploy target, already has own config
- No shared code paths affected

### Failure Modes
| Codepath | Failure | Rescued? | Test? | User Sees |
|----------|---------|-----------|-------|-----------|
| npm install with bad flag | Invalid flag | N/A | No | Vercel build fails immediately |
| JSON syntax error in package.json | Parse error | N/A | No | Vercel build fails immediately |
| JSON syntax error in vercel.json | Parse error | N/A | No | Vercel build fails immediately |

All failures are loud and immediate. No silent failures.

### Completion Summary
```
| Step 0: Scope Challenge   | 2 files, 0 new code — trivial scope        |
| Architecture            | No new arch, ASCII diagram produced          |
| Code Quality            | 0 issues                                   |
| Test Review             | No testable behavior, diagram produced       |
| Performance             | 0 issues (marginal improvement)             |
| NOT in scope           | Written                                    |
| What already exists     | Written                                    |
| Failure modes           | 3 non-critical (all immediate/loud)          |
| Dual voices             | Unavailable (both failed)                   |
| Lake Score              | N/A                                        |
```

---

## Decision Audit Trail

| # | Phase | Decision | Principle | Rationale | Rejected |
|---|-------|----------|-----------|-----------|----------|
| 1 | CEO | HOLD SCOPE | P3 pragmatic | 2-file config, no ambiguity | — |
| 2 | CEO | Approach A (remove) over B/C | P5 explicit | Zero footprint beats extra config | B, C |
| 3 | Eng | No tests needed | P1 completeness | No new codepaths exist | — |

---

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 1 | HOLD | 4 premises valid, 3 approaches eval |
| Codex Review | `codex review` | Independent 2nd opinion | 1 | — | unavailable |
| Eng Review | `/plan-eng-review` | Architecture & tests (required) | 1 | HOLD | 0 issues, 3 non-critical failure modes |
| Design Review | `/plan-design-review` | UI/UX gaps | 0 | Skipped | no UI scope |

**VERDICT:** CEO + ENG CLEARED — ready to implement.
