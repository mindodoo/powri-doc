# Story 2.7.1: Quiz Results Podium & Cosmic Q4 Typography Fix

Status: done

baseline_commit: NO_VCS

## Story

As a **quiz user**,
I want to see my **top 3 resort matches** in a clear podium (featured #1 + compact #2/#3),
So that I get a focused recommendation without scrolling through all 20 resorts.

## Acceptance Criteria

1. **Serious results:** header → podium top 3 immediately (200ms enter) — not full list
2. **Cosmic results:** ~800ms interstitial → header + subline → podium top 3
3. **#1 `QuizTopMatchCard`:** accent border, rank + label, 16/9 hero, badges, `getMatchReason()` line
4. **#2/#3 `QuizRunnerUpRow`:** 48×48 thumb, rank, name, region — no tagline/badges/reason
5. **Browse CTA** below podium → `/explore`
6. **No answer summary** tags under header
7. **`quiz_completed`** after reveal visible; **`result_count: 3`**
8. **Podium taps** → `resort_card_tapped` with `list_context: quiz`, `position` 1/2/3
9. **Cosmic Q4** answer buttons use same DM Sans typography as Q1–Q3 (no italic override)

## Tasks / Subtasks

- [x] **Typography fix** (AC: 9)
  - [x] Remove `COSMIC_EMPHASIZED_OPTION_IDS` and `emphasize` styling from Q4

- [x] **`matchReason.ts` + axis scores** (AC: 3)
  - [x] `computeAxisScores()`, `getMatchReasonKey()` with tie-break order
  - [x] i18n `quiz.results.matchReason.*`

- [x] **Podium components** (AC: 1–4, 8)
  - [x] `QuizTopMatchCard`, `QuizRunnerUpRow`, `scorePodiumResorts()` (top 3)
  - [x] Removed full-list `QuizResults` / `ResortList` on quiz screen

- [x] **`QuizResultsReveal` + analytics timing** (AC: 2, 6, 7)
  - [x] Cosmic 800ms interstitial; `quiz_completed` after reveal visible
  - [x] Browse CTA; no answer summary

- [x] **i18n + validate**
  - [x] Updated intro, podium labels, match reasons, browse CTA
  - [x] `features/quiz.md` UX note aligned (no Q4 italic)
  - [x] `npm run build` + `npm run lint` + `npm run test:quiz`

## Dev Notes

- Primary spec: `features/quiz.md` § Results reveal + § Results podium
- Scoring still ranks all 20 via `scoreResorts()`; `scorePodiumResorts()` slices top 3 for UI
- Serious fires `quiz_completed` on results mount; Cosmic after interstitial

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Replaced full ranked `ResortList` with podium: featured #1 card + minimal #2/#3 rows
- Cosmic oracle interstitial (~800ms) before header/subline/podium
- Match reason on #1 from dominant scoring axis; `quiz_completed` with `result_count: 3`
- Cosmic Q4 answers now use same DM Sans as other questions

### File List

- `_bmad-output/planning-artifacts/features/quiz.md` (modified — Q4 typography note)
- `web/messages/en.json` (modified)
- `web/src/app/globals.css` (modified — podium animation)
- `web/src/lib/quiz/types.ts` (modified — `ResortAxisScores`)
- `web/src/lib/quiz/questions.ts` (modified — removed emphasize)
- `web/src/lib/quiz/score.ts` (modified — axis scores, `scorePodiumResorts`)
- `web/src/lib/quiz/matchReason.ts` (new)
- `web/src/components/quiz/QuizFlow.tsx` (modified)
- `web/src/components/quiz/QuizResultsReveal.tsx` (new)
- `web/src/components/quiz/QuizTopMatchCard.tsx` (new)
- `web/src/components/quiz/QuizRunnerUpRow.tsx` (new)
- `web/src/components/quiz/ResortQuizCard.tsx` (modified)
- `web/src/components/quiz/QuizResults.tsx` (deleted)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All nine ACs satisfied. Quiz results show podium top 3 with Serious/Cosmic reveal flows, match reason on #1, correct analytics timing, Cosmic Q4 typography fix, and persona tests still pass.

### Review Findings

- [x] [Review][Note] `resort_card_tapped` position uses 1-based ranks (1/2/3) on podium vs 0-based on Home list — intentional per story AC
- [x] [Review][Defer] Browse CTA links to `/explore` without `?from=quiz` query param
- [x] [Review][Pass] Persona ranking tests unchanged; podium is slice of same `scoreResorts()` output
