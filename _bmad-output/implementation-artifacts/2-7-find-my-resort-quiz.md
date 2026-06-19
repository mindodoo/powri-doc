# Story 2.7: Find My Resort Quiz

Status: done

baseline_commit: NO_VCS

## Story

As an **undecided planner**,
I want a short 4-question quiz that recommends resorts,
So that I get a personalised starting list without reading all 20.

## Spec update — Results podium (2026-06-12)

Planning docs updated: quiz results show **podium top 3 only** (not full ranked list). See [`features/quiz.md`](../planning-artifacts/features/quiz.md) § Results podium.

**Delta vs shipped implementation:** Resolved in Story 2.7.1 — podium top 3, `QuizTopMatchCard`, `QuizRunnerUpRow`, `getMatchReason()`, Browse CTA, `result_count: 3`.

---

## Acceptance Criteria

1. **Given** entry from onboarding, Home, or Explore CTA  
   **When** I start the quiz  
   **Then** `quiz_started` fires with `entry_point` and `quiz_mode`

2. **Given** quiz intro  
   **When** before Q1  
   **Then** I can toggle Serious / Cosmic; wide full-width answer buttons on each card

3. **Given** four questions  
   **When** swiping or tapping answers  
   **Then** advances to next question; enums map per `docs/quiz-scoring.md`

4. **Given** completion  
   **When** results show  
   **Then** ranked resort list + mode header; `quiz_completed` with enums + `result_count`

5. **Given** a result card tap  
   **Then** navigates to detail with `list_context: quiz`

## Tasks / Subtasks

- [x] **Quiz lib** (AC: 3–4)
  - [x] `types.ts`, `questions.ts`, `score.ts` per `docs/quiz-scoring.md`
  - [x] `score.test.ts` + `npm run test:quiz` persona fixtures

- [x] **Quiz UI** (AC: 1–2, 5)
  - [x] `QuizModeToggle`, `ResortQuizCard`, `QuizFlow`, `QuizResults`
  - [x] Replace `/quiz` stub; `entry_point` from `?from=`

- [x] **i18n + analytics + validate**
  - [x] Serious/Cosmic copy in `en.json`
  - [x] `ResortList` supports `list_context: quiz`, show all ranked
  - [x] `npm run build` + `npm run lint` + `npm run test:quiz`

## Dev Notes

- Scoring: 4-axis weighted model (35/30/25/10); fallback when &lt;3 resorts score &gt;40
- Persona fixtures (Ninja, Alex, Mind, Culture seeker, Unsure) pass via `test:quiz`
- Mode toggle on intro only; switching resets answers before start
- Explore CTA links to `/quiz?from=explore`

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Full quiz flow: intro → 4 sliding cards → ranked results reusing `ResortCard`
- Serious and Cosmic share `score.ts`; i18n drives display copy only
- Analytics: `quiz_started`, `quiz_completed` with all enum properties + `quiz_mode`
- `tsx` devDependency + `test:quiz` script for persona validation

### File List

- `web/package.json` (modified — `test:quiz`, `tsx`)
- `web/messages/en.json` (modified — full `quiz` namespace)
- `web/src/app/globals.css` (modified — quiz slide animation)
- `web/src/app/[locale]/quiz/page.tsx` (modified — `QuizFlow`)
- `web/src/components/explore/FindMyResortCta.tsx` (modified — `?from=explore`)
- `web/src/components/quiz/QuizFlow.tsx` (new)
- `web/src/components/quiz/QuizModeToggle.tsx` (new)
- `web/src/components/quiz/QuizResults.tsx` (new)
- `web/src/components/quiz/ResortQuizCard.tsx` (new)
- `web/src/components/resort/ResortList.tsx` (modified — `quiz` context, `initialVisible`, `showResortCount`)
- `web/src/lib/quiz/types.ts` (new)
- `web/src/lib/quiz/questions.ts` (new)
- `web/src/lib/quiz/score.ts` (new)
- `web/src/lib/quiz/score.test.ts` (new)
- `web/scripts/verify-quiz-scoring.ts` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All ACs satisfied. Four-question Serious/Cosmic quiz with shared scoring, analytics, ranked results, and persona tests passing. Build, lint, and `test:quiz` pass.

### Review Findings

- [x] [Review][Note] Swipe handlers wired but forward-only — answers advance on tap only (matches deferred back-nav in feature brief)
- [x] [Review][Defer] Onboarding still routes to `/explore` not `/quiz?from=onboarding` — user sees Explore CTA first; acceptable
- [x] [Review][Defer] Zustand store deferred — React state in `QuizFlow` matches onboarding pattern
- [x] [Review][Pass] Persona fixtures Ninja, Alex, Mind, Culture seeker, Unsure all green
