# Story 7.1: Quiz In-Page Slide Transitions & Results Spacing

Status: done

baseline_commit: b833449

## Story

As a **quiz player**,  
I want questions to slide horizontally within one screen without the app chrome jumping,  
So that the quiz feels smooth and the results show my #1 match fully on small phones.

## Acceptance Criteria

1. **Given** I am on `/quiz` in the **questions** phase  
   **When** I tap an answer option  
   **Then** the current question **slides out left** and the next **slides in from the right** in a **persistent container** (no full-card remount via `key`)

2. **And** `BottomNav` does not visually flash or jump on answer tap (iPhone SE viewport)

3. **And** progress text updates without layout shift of bottom nav

4. **And** `prefers-reduced-motion: reduce` disables slide animation (instant advance)

5. **And** quiz analytics unchanged (`quiz_started`, `quiz_completed`)

6. **Given** quiz **results** on **375×667**  
   **When** Serious results render  
   **Then** **#1 match card** is fully visible (not top-clipped)

7. **And** results title has **≤24px** top gap below back control (not 48px stacked padding)

## Tasks / Subtasks

- [x] **Slide transition** — timeout-driven exit → advance → enter (no `animationend` dependency)
- [x] **CSS** — `quiz-slide-out-left`, `quiz-slide-in`; reduced-motion media query
- [x] **`ResortQuizCard`** — optional `animationClass`; centered layout preserved (`justify-center`, `py-section-gap`)
- [x] **No `key` remount** on question card
- [x] **Validate** — `test:quiz`, lint, build
- [x] **Results spacing** (AC 6–7) — `pt-4` + compact header gap (7.1b follow-up 2026-06-17)

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- **v1 (reverted):** `animationend`-driven transitions froze quiz when CSS animations did not run.
- **v2 (shipped):** `setTimeout(TRANSITION_MS)` drives exit and enter phases; `isTransitioningRef` blocks double-tap; `prefersReducedMotion()` bypasses animation in JS + CSS.
- Centered question layout unchanged per PO request.
- Results spacing (7.1b): `QuizResultsReveal` uses `pt-4 pb-section-gap`, header `gap-1`, podium block `gap-3` — ~24px below back control on Serious results.

### File List

- `web/src/components/quiz/QuizFlow.tsx`
- `web/src/components/quiz/ResortQuizCard.tsx`
- `web/src/components/quiz/QuizResultsReveal.tsx`
- `web/src/app/globals.css`

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve (AC 1–7 complete after 7.1b follow-up)

### Summary

Question-phase slide transitions ship safely via timeout-based state machine. Results top spacing tightened in 7.1b (`pt-4`, compact header). Centered question layout unchanged.

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass | Exit left + enter right; no `key` on `ResortQuizCard` |
| 2 | Pass | Same `min-h-[calc(100dvh-7rem-…)]` shell; card stays mounted |
| 3 | Pass | Progress row outside animated card; index updates after exit phase (~220ms) |
| 4 | Pass | `prefersReducedMotion()` instant advance + CSS `animation: none` |
| 5 | Pass | `quiz_started` / `quiz_completed` unchanged |
| 6 | Pass | `pt-4` replaces 48px top padding; tighter `gap-3` to podium |
| 7 | Pass | Back `pb-2` (8px) + results `pt-4` (16px) = 24px below back control |

### Review Findings

- [x] [Review][Pass] Timeout fallback (`TRANSITION_MS = 220`) guarantees `finishTransition()` — no permanent lockout
- [x] [Review][Pass] `clearTransitionTimeout` on unmount and mode change prevents leaked timers
- [x] [Review][Pass] `isTransitioningRef` guards `handleAnswer` during exit/enter (~440ms total)
- [x] [Review][Pass] Centered layout preserved: `justify-center`, `py-section-gap`, same button styles
- [x] [Review][Pass] `npm run lint`, `npm run test:quiz`, `npm run build` green
- [x] [Review][Note] Progress counter updates after exit animation, not on tap — acceptable; no bottom-nav layout shift
- [x] [Review][Pass] Results `pt-4 pb-section-gap`, header `gap-1` — AC 6–7 (7.1b follow-up)
- [x] [Review][Note] `touchStartX` ref/handlers still unused (pre-existing) — harmless defer
- [x] [Review][Defer] Overlay back (**7.3**) unchanged — in-flow `QuizBackButton` per Epic 6.4
