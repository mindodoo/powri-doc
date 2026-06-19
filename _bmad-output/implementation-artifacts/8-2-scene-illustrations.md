# Story 8.2: Scene Illustrations (AI + fallback)

Status: done

## Story

As a **Cosmic mode user**,  
I want one illustration per scene that matches the forest path story,  
So that each question feels visual and part of the same world.

## Acceptance Criteria

1. Cosmic Q1–4 show illustration (fork, clearing, gates, stone clock)
2. Consistent style across four assets
3. `web/public/quiz/cosmic/q1.webp` … `q4.webp`
4. Each ≤120KB; total ≤500KB
5. `content/quiz-illustrations/README.md` documents source + prompts

## Tasks

- [x] Generate AI illustrations (PRD §15 style lock)
- [x] Compress to WebP, verify size budget
- [x] Wire `ResortQuizCard` scene image (Cosmic only)
- [x] Update README + sprint-status
- [x] lint / build / test:quiz

## Dev Agent Record

### Agent Model Used

Composer (GenerateImage + sharp post-process)

### Completion Notes

- Four editorial-watercolor scenes generated via Cursor GenerateImage; compressed with **sharp** to WebP 1280×800 (~117/114/114/116 KB; **461 KB total**).
- `sceneIllustrations.ts` maps question index → `/quiz/cosmic/qN.webp`.
- `ResortQuizCard` renders 16:10 band with `next/image` when `sceneIllustrationSrc` set; **Serious unchanged**.
- No Info tab credit (AI in-project generation; documented in README).

### File List

- `web/public/quiz/cosmic/q1.webp` … `q4.webp` (new)
- `web/src/lib/quiz/sceneIllustrations.ts` (new)
- `web/src/components/quiz/ResortQuizCard.tsx`
- `web/src/components/quiz/QuizFlow.tsx`
- `content/quiz-illustrations/README.md`
- `_bmad-output/implementation-artifacts/sprint-status.yaml`

## Senior Developer Review (AI)

**Review date:** 2026-06-18  
**Outcome:** Approve (Q3 Nordic gates PO-approved 2026-06-18; full-set visual sign-off optional before merge)

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass | Illustration band on Cosmic questions only |
| 2 | Pass | Same GenerateImage style anchor for all four |
| 3 | Pass | Assets in `public/quiz/cosmic/` |
| 4 | Pass | All ≤120 KB; total 461 KB |
| 5 | Pass | README updated with tool, prompts, sizes |

### Review Findings

- [x] [Review][Pass] Decorative `alt=""` — question text carries meaning
- [x] [Review][Pass] `priority` on current scene image for LCP
- [x] [Review][Pass] PO approved **Scene 3** Nordic stone arch gates (2026-06-18)
- [x] [Review][Note] Scenes 1/2/4 unchanged from initial 8.2 ship — spot-check on merge if desired
- [x] [Review][Defer] Cosmic button colors, candle, sparkle — Story 8.3
