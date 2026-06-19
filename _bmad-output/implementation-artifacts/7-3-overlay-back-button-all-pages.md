# Story 7.3: Overlay Back Button — All Pages

Status: done

baseline_commit: b833449

## Story

As a **user on any screen**,  
I want the back control fixed at the top-left over the page content,  
So that I can exit consistently without scrolling or losing it to layout shifts.

## Acceptance Criteria

1. Back is `position: fixed` top-left with safe-area + `px-screen-x`
2. `z-index` 45 (above content/top bar 40, below consent 60 / bottom nav 50)
3. Single `OverlayBackButton` replaces `QuizBackButton` + `ResortDetailBack`
4. Quiz → `/explore`; Detail → history back / `/`; Legal → `/info`
5. Content uses `OVERLAY_BACK_OFFSET_CLASS` so titles are not hidden
6. Detail uses `variant="onImage"`; min 44×44 tap target; i18n `aria-label`

## Tasks / Subtasks

- [x] `OverlayBackButton.tsx` + `OVERLAY_BACK_OFFSET_CLASS` in `pageInsets.ts`
- [x] Quiz, Legal, Resort detail wired
- [x] Remove `QuizBackButton.tsx`, `ResortDetailBack.tsx`
- [x] Validate lint + build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- `OverlayBackButton`: `surface` (quiz/legal) and `onImage` (detail) variants; `href` or history+fallback
- Fixed shell uses `pointer-events-none` wrapper + `pointer-events-auto` on control
- `ResortDetailOverlayBack` thin client wrapper for i18n
- Quiz results offset via wrapper; removed redundant `pt-4` on `QuizResultsReveal`
- Legal: icon-only overlay back (text label moved to `aria-label` only)

### File List

- `web/src/components/layout/OverlayBackButton.tsx` (new)
- `web/src/components/resort/ResortDetailOverlayBack.tsx` (new)
- `web/src/lib/layout/pageInsets.ts`
- `web/src/components/quiz/QuizFlow.tsx`
- `web/src/components/quiz/QuizResultsReveal.tsx`
- `web/src/components/legal/LegalPageLayout.tsx`
- `web/src/components/resort/ResortDetailHero.tsx`
- `web/src/app/[locale]/resorts/[slug]/page.tsx`
- Deleted: `QuizBackButton.tsx`, `ResortDetailBack.tsx`

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| Fixed top-left | Pass | `fixed inset-x-0 top-0 z-[45]` |
| z-index layering | Pass | Back 45; top bar 40; bottom nav 50; consent 60 |
| Shared component | Pass | Old back components removed |
| Navigation behaviour | Pass | Quiz link, detail history, legal `/info` |
| Content offset | Pass | `OVERLAY_BACK_OFFSET_CLASS` on quiz/legal |
| onImage + a11y | Pass | `min-h-11 min-w-11`; labels from i18n |

### Review Findings

- [x] [Review][Pass] Quiz all phases share one overlay back + offset wrapper
- [x] [Review][Pass] Detail back stays visible on scroll (fixed onImage over hero)
- [x] [Review][Pass] Hero no longer duplicates absolute back row
- [x] [Review][Pass] `npm run lint`, `npm run build` green
- [x] [Review][Note] Legal back is icon-only now (label in `aria-label`) — cleaner overlay, same destination
