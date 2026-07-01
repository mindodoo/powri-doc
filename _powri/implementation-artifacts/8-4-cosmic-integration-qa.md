# Story 8.4: Cosmic Mode Integration & QA

Status: done

## Story

As a **product owner**,  
I want the full Cosmic experience wired and verified,  
So that production users get the forest path without Serious regressions.

## Acceptance Criteria

1. **Given** `/quiz` Cosmic on iPhone SE (375×667) and iPad/desktop  
   **Then** intro artwork + scene illustrations + progress + sparkle buttons fit without clipping

2. **And** Epic 7.1 slide transitions still work between scenes

3. **And** Cosmic enum answers produce same podium as pre–Epic 8 mapping

4. **And** `npm run lint && npm run build && npm run test:quiz` pass

5. **And** manual smoke checklist in `docs/qa/phase1/manual-qa-epic-08.md` ready for PO pass

6. **And** QA doc reflects **revised 8.3 UX** (fortune intro, white buttons, sparkle-on-tap)

## Tasks

- [x] Verify `QuizFlow` wires intro art, mode toggle, scene illustrations, sparkle, transitions
- [x] Verify `quiz_started` / `quiz_completed` + `story_arc_version: forest-v1c` (Cosmic only)
- [x] Run automated gates: lint, build, test:quiz, test:analytics
- [x] Update `docs/qa/phase1/manual-qa-epic-08.md` for revised 8.3 scope + automated pre-flight
- [x] Confirm cosmic assets deployed under `web/public/quiz/cosmic/`
- [ ] PO manual device matrix + PostHog live verification (deferred to sign-off)

## Dev Agent Record

### Agent Model Used

Composer

### Integration traceability

| Area | Implementation | Verified |
|------|----------------|----------|
| Intro subtitle | `QuizFlow` → `t('modeCosmicSubtitle')` when Cosmic | Code review |
| Intro artwork | `CosmicIntroArtwork` → `cosmic-fortune-start.webp?v=2` | Code review + asset on disk |
| Mode toggle | `QuizModeToggle` + `SparkleIcon` + purple token | Code review |
| Scene progress | `QuizFlow` → `t('sceneProgress')` when Cosmic | Code review |
| Scene illustrations | `getCosmicSceneIllustration` → `ResortQuizCard` | Code review |
| Sparkle on tap | `SparkleAnswerButton` via `sparkleOnTap` prop | Code review |
| Intro → questions transition | `quiz-intro-exit` / `quiz-questions-fade-in` | Code review |
| Results interstitial | `QuizResultsReveal` cosmic loading copy | Code review |
| Analytics | `cosmicStoryArcAnalytics(mode)` on started + completed | test:analytics |
| Scoring unchanged | Same enum keys; Ninja persona | test:quiz |

### Automated gate results (2026-06-18)

```
npm run lint          — pass
npm run build         — pass
npm run test:quiz     — pass (Ninja: rusutsu/kiroro/niseko-united top 3)
npm run test:analytics — pass
```

### Asset budget note

| Set | Size |
|-----|------|
| Q1–Q4 scenes | ~461 KB |
| Intro fortune | ~287 KB |
| **Total** | **~756 KB** |

Original 8.2 AC capped four scenes at ≤500 KB. Intro art is additive; LCP check flagged for manual QA (8.2.3).

### Completion Notes

- Story 8.4 **dev complete**: integration verified, automated gates green, QA doc updated for revised intro UX.
- **PO manual sign-off pending**: device matrix, visual illustration approval, PostHog live events, production smoke.
- Epic release gate (1-week Cosmic vs Serious completion) runs post-deploy — not in this story.

### File List

- `docs/qa/phase1/manual-qa-epic-08.md` (updated)
- `_powri/implementation-artifacts/8-4-cosmic-integration-qa.md` (new)
- `_powri/implementation-artifacts/sprint-status.yaml`

## Senior Developer Review (AI)

**Review date:** 2026-06-18  
**Outcome:** Approve (dev complete; PO manual pass deferred)

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass (code) | Manual clip check on PO pass |
| 2 | Pass (code) | Epic 7 transitions unchanged in `QuizFlow` |
| 3 | Pass | `test:quiz` Ninja persona |
| 4 | Pass | All four scripts green |
| 5 | Pass | QA doc updated |
| 6 | Pass | 8.3 section rewritten for fortune intro + sparkle |
