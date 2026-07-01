# Story 8.3: Cosmic Intro Candle Path, Sparkle & Mode Toggle

Status: done

## Story

As a **Cosmic mode user**,  
I want the magic to begin on the mode-selection intro — candle path, purple toggle, shooting star — while question screens stay clean with sparkle-on-tap only,  
So that Cosmic feels mysterious at the threshold without changing answer chrome on every question.

## Acceptance Criteria

1. Intro: Serious / Cosmic toggle; Cosmic segment shows shooting star icon (muted when inactive)
2. When Cosmic selected: toggle segment → dark purple (`--color-cosmic-purple`); star → yellow-orange (`--color-cosmic-star`)
3. After Cosmic selected: candle path fades in under toggle — large center lantern + smaller dim lanterns (path feel)
4. Start button → dark purple when Cosmic; on click intro fades out (~520ms), questions fade in (~480ms)
5. Question screens: same white/surface answer buttons as Serious; sparkle on tap in Cosmic only (~500ms; off when reduced motion)
6. No floating candle or twilight button colors on question screens
7. Serious mode unchanged

## Tasks

- [x] Cosmic CSS tokens (`--color-cosmic-purple`, star colors)
- [x] `QuizModeToggle` — purple active + `ShootingStarIcon`
- [x] `CosmicIntroCandlePath` + `CosmicLantern` — intro candle path
- [x] `SparkleAnswerButton` — white button + tap sparkle
- [x] `QuizFlow` — intro exit / questions fade-in transitions
- [x] Intro + sparkle animations in `globals.css`
- [x] Remove question-screen `CosmicAnswerButton` / `CosmicCandleAmbience`
- [x] lint / test:quiz / build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- **Intro-only magic:** candle path, purple toggle/start, shooting star accent, fade-out → fade-in transition.
- **Question screens reverted:** standard `bg-surface` bordered buttons; `sparkleOnTap` prop on `ResortQuizCard`.
- **SparkleAnswerButton** — sparkle burst on tap (skipped if reduced motion).
- **CosmicLantern** — reused for intro path; float + flame flicker via CSS.
- Scene illustrations (8.2) unchanged on Cosmic questions.

### File List

- `web/src/styles/tokens.css`
- `web/src/app/globals.css`
- `web/src/components/quiz/ShootingStarIcon.tsx` (new)
- `web/src/components/quiz/CosmicLantern.tsx` (new)
- `web/src/components/quiz/CosmicIntroCandlePath.tsx` (new)
- `web/src/components/quiz/SparkleAnswerButton.tsx` (new)
- `web/src/components/quiz/QuizModeToggle.tsx`
- `web/src/components/quiz/ResortQuizCard.tsx`
- `web/src/components/quiz/QuizFlow.tsx`
- Removed: `CosmicAnswerButton.tsx`, `CosmicCandleAmbience.tsx`

## Senior Developer Review (AI)

**Review date:** 2026-06-18  
**Outcome:** Approve (revised scope — intro-only candle path)

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass | Shooting star on Cosmic toggle; muted when inactive |
| 2 | Pass | Purple active segment + yellow-orange star |
| 3 | Pass | 5-lantern path; fade-in 650ms; dim smaller candles behind center |
| 4 | Pass | Purple start button; intro exit 520ms; questions fade-in 480ms |
| 5 | Pass | White surface buttons + sparkle; reduced motion respected |
| 6 | Pass | No candle or colored buttons on questions |
| 7 | Pass | Serious unchanged |

### Review Findings

- [x] [Review][Pass] Design tokens align with Theme A framework
- [x] [Review][Pass] Candle path `aria-hidden`; decorative only
- [x] [Review][Defer] Story 8.4 — full Epic 8 QA doc + manual smoke
