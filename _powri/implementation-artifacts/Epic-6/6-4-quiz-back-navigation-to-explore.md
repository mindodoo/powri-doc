# Story 6.4: Quiz Back Navigation to Explore

Status: done

baseline_commit: 468a56f

## Story

As a **user deep in the quiz**,  
I want a back control that returns me to Explore,  
So that I can exit the quiz flow without using browser back or bottom nav alone.

## Acceptance Criteria

1. **Given** I am on `/quiz` in **intro**, **questions**, or **results** phase  
   **When** the screen renders  
   **Then** a **back arrow** appears top-left

2. **When** I tap back  
   **Then** I navigate to **`/explore`**

3. **And** back control respects safe-area inset top

4. **And** `aria-label` from i18n (`quiz.backToExplore`)

5. **And** results phase (including Cosmic interstitial) shows back before results content

## Tasks / Subtasks

- [x] **`QuizBackButton`** — Link to `/explore`, ArrowLeft, 44px tap target
- [x] **Wire in `QuizFlow`** — intro, questions, results wrapper
- [x] **i18n** — `quiz.backToExplore`
- [x] **Validate** — quiz tests, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Light-surface pill button (border + surface bg) — distinct from hero `ResortDetailBack` dark overlay
- Direct `/explore` link (not `router.back()`) per AC
- `quiz_abandoned` analytics deferred per epic

### File List

- `web/src/components/quiz/QuizBackButton.tsx` (new)
- `web/src/components/quiz/QuizFlow.tsx`
- `web/messages/en.json`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

Back control present on all quiz phases including results wrapper; navigates deterministically to Explore with accessible label and safe-area padding.

### Review Findings

- [x] [Review][Pass] Back visible on intro, questions, results (incl. Cosmic interstitial via wrapper)
- [x] [Review][Pass] Links to `/explore`; `aria-label` from i18n
- [x] [Review][Pass] `min-h-11 min-w-11` tap target; safe-area top padding
- [x] [Review][Pass] quiz tests, lint, build green
- [x] [Review][Note] No `quiz_abandoned` event (deferred per epic)
