# Story 8.1: Snow Forest Path Copy & Cosmic Subtitle

Status: done

baseline_commit: (main @ Epic 7 done)

## Story

As a **quiz taker choosing Cosmic**,  
I want a clear subtitle and the **forest-v1c** story copy across four scenes,  
So that I understand what Cosmic means and stay engaged to the end.

## Acceptance Criteria

1. **Given** the quiz intro  
   **When** Cosmic is selected  
   **Then** subtitle shows: *Let the universe choose for me*

2. **And** Cosmic progress reads **Scene {n} of 4** (Serious keeps Question {n} of 4)

3. **And** Cosmic questions/options match PRD §6 **forest-v1c**

4. **And** Cosmic results interstitial, header, and subline match PRD §8

5. **And** `quiz_started` / `quiz_completed` include `story_arc_version: "forest-v1c"` when `quiz_mode: cosmic` only

6. **And** Serious mode copy and analytics unchanged

7. **And** `npm run test:quiz`, `npm run test:analytics`, lint, build pass

## Tasks / Subtasks

- [x] **CS** — Story file + sprint-status
- [x] **i18n** — `web/messages/en.json` forest-v1c + subtitle + scene progress + results
- [x] **UI** — Cosmic subtitle on intro; scene progress in questions phase
- [x] **Analytics** — `story_arc_version`, tracking-plan, phase1Events notes
- [x] **DS** — Verify scripts green
- [x] **CR** — Senior dev review + AC traceability

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Shipped **forest-v1c** Cosmic copy in `en.json` (Snow Forest Path + stone clock Scene 4).
- Intro shows **modeCosmicSubtitle** when Cosmic selected; questions use **sceneProgress** for Cosmic only.
- `cosmicStoryArcAnalytics()` centralises `story_arc_version: forest-v1c` on quiz events (Cosmic only).
- Results strings updated per PRD §8 (interstitial, header, subline).
- Serious strings and scoring enums unchanged; persona tests pass.

### File List

- `web/messages/en.json`
- `web/src/lib/quiz/storyArc.ts` (new)
- `web/src/components/quiz/QuizFlow.tsx`
- `web/src/components/quiz/QuizResultsReveal.tsx`
- `web/src/lib/analytics/phase1Events.ts`
- `docs/analytics/tracking-plan.md`
- `_powri/implementation-artifacts/sprint-status.yaml`

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve

### Summary

Copy-only story with minimal UI wiring. Enum keys unchanged; scoring personas unaffected. Analytics property scoped to Cosmic via shared helper — no leakage on Serious events.

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass | Subtitle renders when `mode === 'cosmic'` on intro |
| 2 | Pass | `sceneProgress` vs `progress` by mode in questions phase |
| 3 | Pass | All four scenes match PRD §6 forest-v1c |
| 4 | Pass | `reveal.cosmic.loading`, `results.cosmic`, `results.cosmicSubline` updated |
| 5 | Pass | `story_arc_version` spread only when `mode === 'cosmic'` |
| 6 | Pass | Serious i18n + no extra analytics props on serious |
| 7 | Pass | lint, test:quiz, test:analytics, build green |

### Review Findings

- [x] [Review][Pass] Option `id` keys unchanged — scoring path stable
- [x] [Review][Pass] `COSMIC_STORY_ARC_VERSION` single constant for future arc bumps
- [x] [Review][Pass] tracking-plan + phase1Events notes document optional property
- [x] [Review][Note] Thai locale N/A (EN-only scaffold); Cosmic English OK per PRD
- [x] [Review][Defer] Illustrations, candle, sparkle buttons — Stories 8.2–8.3
