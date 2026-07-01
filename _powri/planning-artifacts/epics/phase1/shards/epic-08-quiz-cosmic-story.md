<!-- generated from epics/phase1/epics-phase1.md — do not edit; run prd-shard-regen.md -->

# Epic 8: Quiz Cosmic Mode UX v2

**Status:** Backlog · **Phase:** 1.3 · **Runs after:** Epic 7 on `main`  
**Goal:** Snow Forest Path narrative, scene illustrations, cosmic buttons + sparkles, floating candle on light page.

> **AI agents:** Read this shard + [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../../prds/phase1/shards/18-quiz-cosmic-story-ux.md). One story per session. **Do not change `score.ts` enums or weights.**

---

## Overview

| Type | Stories | Description |
|------|---------|-------------|
| **Content / copy** | 8.1 | Snow Forest Path copy, Cosmic subtitle, results strings, analytics property |
| **Visual / assets** | 8.2 | Four AI scene illustrations (+ licensed fallback if needed) |
| **UI / motion** | 8.3 | Cosmic button palette, sparkle tap, floating candle SVG |
| **Integration** | 8.4 | Wire into `QuizFlow` / `ResortQuizCard`; QA + attribution |

**Out of this epic:** Serious mode changes, scoring algorithm, dark cosmic stage, sound, Thai Cosmic copy.

---

## Requirements map

| ID | Source | Story |
|----|--------|-------|
| PO-13 | Cosmic unclear; needs subtitle | **8.1** |
| PO-14 | Magic buttons + sparkle | **8.3** |
| PO-15 | One illustration per question, same style | **8.2** |
| PO-16 | Snow Forest Path storyline | **8.1** |
| PO-17 | Light page + floating candle (not dark stage) | **8.3**, **8.4** |
| PO-18 | AI illustrations; free licensed fallback OK | **8.2** |

**PRD:** [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../../prds/phase1/shards/18-quiz-cosmic-story-ux.md)

---

## PO decisions (reference)

- **Label:** Cosmic + subtitle *Let the universe choose for me*
- **Arc:** Snow Forest Path (Option A)
- **Background:** Light `--color-bg` — no dark stage in v1
- **Ambience:** Floating candle animation near question
- **Assets:** AI locked style primary; Storyset/unDraw/CC0 fallback

---

## Recommended implementation order

```
8.1 (copy + i18n + story_arc_version) ──┐
8.2 (illustrations — parallel) ──────────┼──► 8.4 (integration + QA)
8.3 (tokens + sparkle + candle) ─────────┘
```

---

## Story 8.1: Snow Forest Path Copy & Cosmic Subtitle {#story-81}

**Type:** `[content]` · **FR:** FR-4d, FR-4e, FR-4g, FR-4j · **Depends on:** none

As a **quiz taker choosing Cosmic**,  
I want a clear subtitle and a connected forest story across four scenes,  
So that I know what Cosmic means and stay engaged to the end.

### Acceptance criteria

**Given** the quiz intro  
**When** Cosmic is selected or visible on the toggle  
**Then** subtitle shows: *Let the universe choose for me* (`quiz.modeCosmicSubtitle`)

**And** Cosmic questions use PRD §6 **`forest-v1c`** copy

**And** Cosmic progress reads **Scene {n} of 4** (`quiz.sceneProgress`)

**And** results strings match PRD §8

**And** `quiz_started` / `quiz_completed` include `story_arc_version: "forest-v1c"`

**And** option `id` keys unchanged — `npm run test:quiz` + Ninja persona pass

### Files (expected)

- `web/messages/en.json`
- `docs/analytics/tracking-plan.md` — document `story_arc_version`
- `web/src/lib/analytics` or quiz track calls — add property

---

## Story 8.2: Scene Illustrations (AI + fallback) {#story-82}

**Type:** `[content]` · **FR:** FR-4a · **Depends on:** none (PO style QC)

As a **Cosmic mode user**,  
I want one illustration per scene that matches the forest path story,  
So that each question feels visual and part of the same world.

### Acceptance criteria

**Given** Cosmic questions 1–4  
**Then** each shows illustration per PRD §7.6 (fork, clearing, gates, stone clock)

**And** all four pass style QC: same medium, line weight, palette

**And** assets at `web/public/quiz/cosmic/q1.webp` … `q4.webp`

**And** each **≤120KB** WebP; total **≤500KB**

**And** [`content/quiz-illustrations/README.md`](../../../../../content/quiz-illustrations/README.md) records source, prompts, license, attribution

**When** AI batch fails consistency after 2 rounds  
**Then** fallback to free licensed set (Storyset / unDraw / CC0) per PRD §7.7

**And** Info tab credit added if license requires (`info.illustrationsCredit`)

---

## Story 8.3: Cosmic Buttons, Sparkle & Floating Candle {#story-83}

**Type:** `[cosmetic]` · **FR:** FR-4b, FR-4c, FR-4h, FR-4i · **Depends on:** none

As a **Cosmic mode user**,  
I want colored magical buttons and a gentle floating candle near the question,  
So that the mode feels mystical while the page stays clean and light.

### Acceptance criteria

**Given** Cosmic mode on question screens  
**Then** page background remains `--color-bg` (light)

**And** four answer buttons use `--color-cosmic-opt-1` … `opt-4`

**When** I tap a button  
**Then** sparkle animation plays (~400–600ms) unless reduced motion

**And** `CosmicCandleAmbience` SVG floats + flickers near question (PRD §7.5)

**When** `prefers-reduced-motion: reduce`  
**Then** no sparkle, no candle motion (static candle OK)

**Given** Serious mode  
**Then** unchanged white buttons; no candle; standard progress label

**And** button text contrast ≥ WCAG AA (4.5:1)

### Files (expected)

- `web/src/styles/tokens.css`
- `web/src/components/quiz/CosmicCandleAmbience.tsx`
- `web/src/components/quiz/ResortQuizCard.tsx` — `variant="cosmic"`
- Sparkle styles (component-scoped CSS or `globals.css`)

---

## Story 8.4: Cosmic Mode Integration & QA {#story-84}

**Type:** `[feature]` · **FR:** FR-4a–FR-4j · **Depends on:** 8.1, 8.2, 8.3

As a **product owner**,  
I want the full Cosmic experience wired and verified,  
So that production users get the forest path without Serious regressions.

### Acceptance criteria

**Given** `/quiz` Cosmic on iPhone SE (375×667) and iPad/desktop  
**Then** illustration + scene progress + candle + buttons fit without clipping

**And** Epic 7.1 slide transitions still work between scenes

**And** Cosmic enum answers produce same podium as pre–Epic 8 mapping

**And** `npm run lint && npm run build && npm run test:quiz` pass

**And** manual smoke: intro subtitle → 4 scenes → interstitial → podium → detail tap

**And** [`docs/qa/phase1/manual-qa-epic-08.md`](../../../../../docs/qa/phase1/manual-qa-epic-08.md) checklist completed

---

## Analytics

| Event | New property |
|-------|----------------|
| `quiz_started` | `story_arc_version: "forest-v1c"` when `quiz_mode: cosmic` |
| `quiz_completed` | same |

Update [`docs/analytics/tracking-plan.md`](../../../../../docs/analytics/tracking-plan.md) in Story 8.1.

---

## Release gate

- Stories 8.1–8.4 complete
- Illustration attribution on Info if required
- PostHog: Cosmic vs Serious completion compared 1 week post-deploy
