# Story 6.3: Quiz Podium #1 — Rank Overlay on Image

Status: done

baseline_commit: 2a59f80

## Story

As a **quiz completer**,  
I want the #1 result to look like a clear winner without a side label column,  
So that the card feels visual-first like other resort cards.

## Acceptance Criteria

1. **Given** quiz results with `QuizTopMatchCard` for rank #1  
   **When** results render (Serious or Cosmic)  
   **Then** the left side column with **"1"** and **"Your top match"** text is **removed**

2. **And** a **"1"** badge overlays the **top-left** of the resort image

3. **And** badge is readable on photo and placeholder (accent pill, 32×32px min)

4. **And** full-width single column: image, region, name, match reason, tagline, badges, chevron

5. **And** `resort_card_tapped` with `position: 1`, `list_context: quiz` unchanged

6. **And** unused i18n keys `topMatchLabel` / `topMatchLabelCosmic` removed

## Tasks / Subtasks

- [x] **Redesign `QuizTopMatchCard`** — image overlay rank badge; remove side rail
- [x] **Remove `mode` prop** from card (labels no longer on card)
- [x] **Clean i18n** — remove `quiz.results.topMatchLabel*`
- [x] **Validate** — quiz tests, analytics, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Rank badge: accent background, `text-on-accent`, top-left `absolute` on image container
- Mode-specific copy remains on results **header** only (`quiz.results.serious` / `cosmic`)

### File List

- `web/src/components/quiz/QuizTopMatchCard.tsx`
- `web/src/components/quiz/QuizResultsReveal.tsx`
- `web/messages/en.json`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

Cosmetic-only change. Card layout now matches standard full-width resort card pattern; rank communicated via image overlay. Analytics and quiz scoring untouched.

### Review Findings

- [x] [Review][Pass] Side column removed; overlay badge on image
- [x] [Review][Pass] `mode` prop dropped from `QuizTopMatchCard`; header still mode-aware
- [x] [Review][Pass] Unused i18n keys removed
- [x] [Review][Pass] `test:quiz`, `test:analytics`, lint, build green
- [x] [Review][Defer] Update `features/quiz/brief.md` podium spec when next editing quiz brief
