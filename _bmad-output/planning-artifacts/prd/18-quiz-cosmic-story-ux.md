# PRD §3.10: Quiz Cosmic Mode UX v2 (Epic 8)

**Created:** 2026-06-17 · **Updated:** 2026-06-17  
**Status:** PO decisions locked — ready for implementation  
**Source:** Product owner feedback post–Epic 7 launch  
**Epic shard:** [`../epics/epic-08-quiz-cosmic-story.md`](../epics/epic-08-quiz-cosmic-story.md)  
**Feature brief (baseline):** [`../features/quiz.md`](../features/quiz.md)  
**Asset guide:** [`../../../content/quiz-illustrations/README.md`](../../../content/quiz-illustrations/README.md)  
**Timing:** After Epic 7 on `main`. Serious mode unchanged unless noted.

---

## 0. Document purpose

This PRD defines the **Cosmic mode refresh** for the Find My Resort Quiz: the **Snow Forest Path** story arc, per-question illustrations, mystical answer buttons with sparkle motion, and a **floating candle ambience** on the existing light app background. Scoring enums and `score.ts` stay **unchanged** — only display copy, visuals, and Cosmic-only chrome ship here.

---

## 0.1 Product decisions (locked)

| Topic | Decision | Notes |
|-------|----------|-------|
| **Mode label** | Keep **Cosmic** | Subtitle under toggle when Cosmic selected: *“Let the universe choose for me”* |
| **Story arc** | **Option A — Snow Forest Path** | Four linked scenes; copy in §6 |
| **Page background** | **Light** (`--color-bg` / `#f7f5f2`) | No full dark cosmic stage in v1; keeps app clean and consistent |
| **Ambient motion** | **Floating lighted candle** near question block | SVG/CSS; gentle float + flicker; forest-lantern motif |
| **Scene illustrations** | **Static WebP**, one per question | AI-generated with locked style prompt (primary); free licensed set as fallback |
| **Attribution** | **Required** where license demands | Credits in Info tab + `content/quiz-illustrations/README.md` |
| **Analytics enum** | **`quiz_mode: cosmic`** unchanged | `story_arc_version: "forest-v1c"` on quiz events |
| **Deferred** | Full dark cosmic stage | PO may revisit; do not implement in Epic 8 |

---

## 1. Problem statement

| Issue | Evidence | Impact |
|-------|----------|--------|
| **“Cosmic” is opaque** | Toggle label alone doesn’t explain the mode | Users skip Cosmic or pick it without knowing what to expect |
| **Questions feel disconnected** | Q1 = phone call; Q2 = evening; Q3 = postcard; Q4 = bookstore | No progression; hard to stay engaged |
| **Visually flat** | Same white buttons as Serious; no imagery ([`features/quiz.md`](../features/quiz.md) § UX notes) | Cosmic feels like a text swap |
| **Weak ski-trip link** | Q1 park/phone metaphor doesn’t evoke Japan winter travel | Playful mode under-delivers on trip planning |

**Serious mode** remains the default, direct path. This epic only elevates **Cosmic**.

---

## 2. Vision

Cosmic mode becomes a **short illustrated winter fable** — *The Snow Forest Path* — four scenes, one decision each, same scoring underneath. The page stays **clean and light** like the rest of the app; magic comes from **scene art**, **twilight-colored answer buttons with sparkles**, and a **soft floating candle** that drifts beside the question like a forest lantern guiding the way.

Users who want the universe to choose get a memorable, slightly mystical journey; users who want speed keep Serious.

---

## 3. Target user

### Jobs to be done

- **Undecided planner (Alex):** “Make the quiz fun enough that I finish it instead of bouncing to filters.”
- **Repeat visitor (Mind):** “Give me a reason to try Cosmic after Serious.”
- **Cosmic-curious user:** “I don’t know what Cosmic means — the subtitle should tell me it’s hands-off / fun.”

### User journey — UJ-8. The forest path

- **Persona + context:** First-time visitor, tapped “Find my resort” from Home.
- **Entry state:** `/quiz` intro; **Serious** (default) and **Cosmic** with subtitle *Let the universe choose for me*.
- **Path:** Chooses Cosmic → Scene 1 illustration + fork question + candle ambience → colored sparkle buttons → slide to scenes 2–4 → path-reading interstitial → podium top 3.
- **Climax:** Results header ties back to the forest (“The path led you here…”).
- **Resolution:** Taps #1 resort or Browse all → Explore.
- **Edge case:** `prefers-reduced-motion: reduce` → no sparkles, no candle motion; static candle icon optional.

---

## 4. Glossary

- **Cosmic mode** — User-facing label and internal enum `quiz_mode: cosmic` (unchanged in PostHog).
- **Snow Forest Path** — Approved story arc (`story_arc_version: forest-v1c`); four scenes in §6.
- **Scene illustration** — One static image per question; same visual style across all four.
- **Candle ambience** — Decorative floating lantern/candle SVG near the question; CSS animation only.
- **Sparkle layer** — Particle shimmer on Cosmic answer button tap; decorative only.
- **Scoring enum** — `level`, `priority`, `region`, `timing` — unchanged; see [`docs/quiz-scoring.md`](../../../docs/quiz-scoring.md).

---

## 5. Narrative — Snow Forest Path (approved)

**Premise:** You wander into a quiet snow forest near the Japanese mountains. Each question is the next step on the path.

| Q | Axis | Scene | Narrative beat |
|---|------|-------|----------------|
| 1 | `level` | Fork in the trail | Which slope/path matches your comfort |
| 2 | `priority` | Valley clearing at dusk | What makes the perfect evening |
| 3 | `region` | Three spirit gates in the snow | Which regional gate calls to you |
| 4 | `timing` | Stone winter clock in a clearing | When your journey begins |

### Alternatives considered (not shipping)

| Option | Why not chosen |
|--------|----------------|
| B — Powder Oracle’s Cabin | Indoor scenes risk visual sameness |
| C — Night Train North | Weaker Q1 skill metaphor |
| Rename Cosmic → Story | PO prefers **Cosmic** + subtitle |
| Full dark cosmic stage | PO prefers light, clean page; deferred |

---

## 6. Copy — Snow Forest Path

Implement in `web/messages/en.json` under `quiz.cosmic.*`. Option `id` keys **unchanged** (scoring stable).

**Copy principle:** Answer choices stay inside the forest fable — not ski jargon. Each option **implies** the scoring enum without naming runs or slopes.

**Status:** **Locked** — PO mixed A/B selections (2026-06-17). Ship **`forest-v1c`** below. A/B drafts kept in §6.1 for reference only.

### Mode toggle & intro

| Element | Copy | i18n key |
|---------|------|----------|
| Mode label | Cosmic | `quiz.modeCosmic` (existing) |
| Mode subtitle (Cosmic active) | Let the universe choose for me | `quiz.modeCosmicSubtitle` |
| Cosmic intro line (optional, below toggle) | Follow the snow forest path — four scenes, one resort match. | `quiz.introCosmic` |
| Scene progress | Scene {current} of {total} | `quiz.sceneProgress` (Cosmic only; Serious keeps `quiz.progress`) |

---

### Final copy — `forest-v1c` *(PO approved · ship in 8.1)*

#### Scene 1 — `level` (fork in the trail)

**Question:** *You reach a fork in the snow-laced trail. A wooden sign reads: “Choose your path.”*

| Option id | Copy |
|-----------|------|
| `beginner` | You take the lantern-lit path — slow and easy. |
| `confident` | The worn path looks safe; you'll try it and see how it goes. |
| `advanced` | A narrow opening in the pines — only you noticed it — and you slip through. |
| `group` | You wait at the sign for the others — you'll follow their lead. |

#### Scene 2 — `priority` (valley at dusk)

**Question:** *The forest opens to a valley. What would make tonight perfect?*

| Option id | Copy |
|-----------|------|
| `powder_terrain` | You stay out in the quiet snow — just you and the mountains. |
| `village_culture` | You walk toward the village lights, food, and narrow streets. |
| `family` | Smoke rises from a hut where voices sound safe — you head toward the fire. |
| `all` | Why choose one ending? You'll take the long way and have a little of everything. |

#### Scene 3 — `region` (spirit gates)

**Question:** *Three gates stand in the snow. Which one pulls you through?*

| Option id | Copy |
|-----------|------|
| `hokkaido` | The far gate — wide open snow and cold northern wind. |
| `honshu` | The middle gate hums like distant rails; steam curls up from a bowl you can't see yet. |
| `tohoku` | The northern gate breathes hot spring mist against the cold air. |
| `unsure` | You don't walk through any gate — you're not ready to choose. |

#### Scene 4 — `timing` (stone winter clock)

**Question:** *You find a stone clock in the snow. Four faces mark different depths of winter. When does your journey begin?*

> Replaces “frozen sundial” — easier to picture; choice copy unchanged (still uses *face* / *hand*).

| Option id | Copy |
|-----------|------|
| `early_season` | The face of thin light — early footsteps, an empty world still waking. |
| `peak_february` | The face locked in deepest winter — the hour when everything holds still. |
| `late_season` | The face where snow softens and the shadows grow long. |
| `still_planning` | The hand keeps turning — you'll know when it finally stops. |

**Analytics:** `story_arc_version: "forest-v1c"` on `quiz_started` / `quiz_completed` when `quiz_mode: cosmic`.

---

### §6.1 Draft variants (reference only — do not ship)

<details>
<summary>Variant A / B drafts used during PO review</summary>

Questions identical except Scene 4 (v1c question above).

#### Scene 1 — `level` (fork in the trail)

**Question:** *You reach a fork in the snow-laced trail. A wooden sign reads: “Choose your path.”*

| Option id | Copy (A) |
|-----------|----------|
| `beginner` | The lantern path dims and brightens slowly — you trust its unhurried glow. |
| `confident` | One trail looks well-walked; you'll follow it once before going further. |
| `advanced` | A narrow opening in the pines — only you noticed it — and you slip through. |
| `group` | You linger at the sign until footsteps catch up behind you. |

#### Scene 2 — `priority` (valley at dusk)

**Question:** *The forest opens to a valley. What would make tonight perfect?*

| Option id | Copy (A) |
|-----------|----------|
| `powder_terrain` | You keep walking until the forest goes quiet and the day settles in your bones. |
| `village_culture` | Lantern-light flickers between distant roofs — you drift toward the warmth and broth-steam. |
| `family` | Smoke rises from a hut where voices sound safe — you head toward the fire. |
| `all` | Why choose one ending? You'll take the long way and have a little of everything. |

#### Scene 3 — `region` (spirit gates)

**Question:** *Three gates stand in the snow. Which one pulls you through?*

| Option id | Copy (A) |
|-----------|----------|
| `hokkaido` | The farthest gate — open white silence and a distant cowbell on the wind. |
| `honshu` | The middle gate hums like distant rails; steam curls up from a bowl you can't see yet. |
| `tohoku` | The northern gate breathes hot spring mist against the cold air. |
| `unsure` | None of the gates feel ready — you wait in the grey between them. |

#### Scene 4 — `timing` (frozen sundial)

**Question:** *A frozen sundial shows four faces. When does your journey begin?*

| Option id | Copy (A) |
|-----------|----------|
| `early_season` | The face of thin light — early footsteps, an empty world still waking. |
| `peak_february` | The face locked in deepest winter — the hour when everything holds still. |
| `late_season` | The face where snow softens and the shadows grow long. |
| `still_planning` | The hand keeps turning — you'll know when it finally stops. |

---

### Variant B — Clear story *(recommended · easier to read · still Cosmic)*

Same four questions. Shorter words; each choice reads as a plain story action.

#### Scene 1 — `level`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `beginner` | You take the lantern-lit path — slow and easy. | Just starting / gentle |
| `confident` | The worn path looks safe; you'll try it and see how it goes. | Getting comfortable |
| `advanced` | You spot a hidden gap in the trees and go through on your own. | Bold / experienced |
| `group` | You wait at the sign for the others — you'll follow their lead. | Mixed group |

#### Scene 2 — `priority`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `powder_terrain` | You stay out in the quiet snow — just you and the mountains. | Powder & terrain |
| `village_culture` | You walk toward the village lights, food, and narrow streets. | Village & culture |
| `family` | You join the group by a warm fire — everyone together tonight. | Family-friendly |
| `all` | You want a bit of everything — snow, town, and good company. | All of the above |

#### Scene 3 — `region`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `hokkaido` | The far gate — wide open snow and cold northern wind. | Hokkaido |
| `honshu` | The middle gate — big mountains and a train far in the distance. | Honshu |
| `tohoku` | The gate in rising steam, like a hot spring town in winter. | Tohoku |
| `unsure` | You don't walk through any gate — you're not ready to choose. | Not sure yet |

#### Scene 4 — `timing`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `early_season` | Early winter — the season is just starting, not many people around. | Dec–Jan |
| `peak_february` | Mid-winter — deep snow and the busiest time on the mountain. | Peak Feb |
| `late_season` | Late winter — softer snow and warmer, longer days. | Mar–Apr |
| `still_planning` | The dial still spins — you haven't picked your dates yet. | Still planning |

</details>

---

## 7. Visual design — Cosmic chrome (light page)

Applies **only** when `quiz_mode === cosmic`. **Page background stays `--color-bg`** (#f7f5f2). Serious mode unchanged.

### 7.1 Layout — `ResortQuizCard` (Cosmic)

```
┌─────────────────────────────────────┐  ← --color-bg (light, unchanged)
│  [Scene illustration 16:10]         │
│  Scene 2 of 4                       │
├─────────────────────────────────────┤
│      🕯️ ← floating candle (ambient) │
│  Question (serif heading)           │
│  ┌───────────────────────────────┐  │
│  │ Answer A (twilight violet)  ✨│  │
│  ├───────────────────────────────┤  │
│  │ Answer B (aurora teal)        │  │
│  …                                  │
└─────────────────────────────────────┘
```

- Illustration band: full content width, rounded `--radius-card`, subtle shadow optional
- Candle: positioned **upper-right or left of question** (implementation picks one; avoid overlapping question text on 375px)
- Serious: no illustration, no candle, standard progress copy

### 7.2 Cosmic palette (CSS tokens)

Add to `tokens.css` under `[data-theme='default']`:

| Token | Hex | Use |
|-------|-----|-----|
| `--color-cosmic-opt-1` | `#4a3f8c` | Answer 1 — twilight violet |
| `--color-cosmic-opt-2` | `#2d6b8a` | Answer 2 — aurora teal |
| `--color-cosmic-opt-3` | `#6b4a7a` | Answer 3 — dusk plum |
| `--color-cosmic-opt-4` | `#8a6b3a` | Answer 4 — star gold |
| `--color-cosmic-sparkle` | `#e8e0ff` | Sparkle particle highlight |
| `--color-cosmic-candle-glow` | `#f5d78e` | Candle flame / glow |
| `--color-cosmic-on-opt` | `#ffffff` | Button label on colored buttons |

**Not used in v1:** `--color-cosmic-bg` dark stage tokens (reserved for future experiment).

### 7.3 Answer buttons

| Aspect | Serious (unchanged) | Cosmic |
|--------|---------------------|--------|
| Layout | Full-width, min 52px | Same |
| Background | `surface` + divider border | `--color-cosmic-opt-N` by option index (1–4) |
| Text | `text-primary` | `--color-cosmic-on-opt` |
| Border | 1px divider | None or 1px rgba white 20% |
| Press | `scale(0.99)` | Same + sparkle burst |

Map colors by **option order** (not enum value) for visual balance every question.

### 7.4 Sparkle animation

| Rule | Spec |
|------|------|
| Trigger | On `:active` / tap |
| Technique | CSS pseudo-elements or inline SVG — **no Lottie in v1** |
| Duration | 400–600ms ease-out |
| Particles | 6–10 points along button top edge; fade + translate |
| Reduced motion | Disabled entirely; solid color press state only |
| Performance | `transform` + `opacity` only |

### 7.5 Floating candle ambience

| Rule | Spec |
|------|------|
| Asset | Inline **SVG** lantern or candle (single component `CosmicCandleAmbience`) |
| Placement | Near question heading; `position: absolute` within question block wrapper |
| Float | Slow vertical drift ±6px, ~4s loop, ease-in-out |
| Flicker | Flame opacity 85–100%, ~1.2s irregular loop (two keyframes enough) |
| Size | ~32–40px wide; must not reduce question tap area |
| Reduced motion | Static SVG at rest position; no animation |
| z-index | Below back button; above background only |

Candle reinforces forest-lantern motif from Scene 1 without darkening the page.

### 7.6 Scene illustrations

| # | Scene | Visual brief |
|---|-------|--------------|
| 1 | **The fork** | Snow trail splits; lantern on left path; darker trees on right; no people |
| 2 | **The clearing** | Wide valley, dusk sky, distant village lights vs open snow |
| 3 | **The gates** | Three Nordic stone arch gates (carved rock, Frozen 2–inspired); subtle regional hints (fields / train / steam) |
| 4 | **The stone clock** | Old stone clock in snow; four faces suggesting early / deep / late winter + turning hand |

**Style direction (locked across all four):**

- Medium: soft gouache / editorial watercolor — **not** anime, **not** 3D, **not** photoreal
- Palette: muted winter — cream `#f7f5f2`, slate blue, pine green, touch of `#c4873a` (accent-warm)
- Line: consistent ink weight; no text in image
- Format: **WebP**, 800×500 logical (~1600×1000 export), **≤120KB** each
- Path: `web/public/quiz/cosmic/q1.webp` … `q4.webp`
- Delivery: `next/image` with `priority` on current scene only

### 7.7 Illustration sourcing (locked)

**Primary — AI with locked style**

1. Write one **style anchor prompt** (§15) and reuse for all four scenes; vary **scene prompt** only.
2. Generate 2–3 candidates per scene; PO picks one set where all four feel same “artist.”
3. Post-process: crop, compress WebP, verify file size budget.
4. Document model + prompts in [`content/quiz-illustrations/README.md`](../../../content/quiz-illustrations/README.md).

**Fallback — free licensed illustration set**

Use only if AI batch fails consistency QC after 2 rounds:

| Source | License | Cost | Notes |
|--------|---------|------|-------|
| [Storyset](https://storyset.com) | Free with attribution | $0 | Recolor to brand palette |
| [unDraw](https://undraw.co) | Free, no attribution required | $0 | May feel generic; customize colors |
| [Open Doodles](https://opendoodles.com) | CC0 | $0 | Looser style; harder to match Kinfolk tone |

**Attribution:** If required, add **Illustrations** line on Info tab (`info.illustrationsCredit`) linking to artist/source. Mirror full credits in `content/quiz-illustrations/README.md`.

---

## 8. Results reveal (Cosmic)

Enum/scoring unchanged. Update strings only:

| Step | Was | Now |
|------|-----|-----|
| Interstitial | Consulting the powder oracle… | *Reading the signs on the path…* |
| Header | The stars have spoken. Here are your resorts: | *The path led you here — your top resorts:* |
| Subline | Ranked by cosmic vibes and extremely real resort data. | *Chosen by the universe — backed by real resort data.* |
| #1 label | Your cosmic #1 | *Your cosmic #1* (keep — label still fits PO tone) |

i18n keys: `quiz.reveal.cosmic.loading`, `quiz.results.cosmic`, `quiz.results.cosmicSubline`, `quiz.results.topMatchLabelCosmic`.

Interstitial timing: **~800ms** crossfade (unchanged).

---

## 9. Functional requirements

| ID | Requirement |
|----|-------------|
| **FR-4a** | Cosmic shows **one scene illustration per question** (4 assets), consistent style |
| **FR-4b** | Cosmic answer buttons use **four distinct palette colors** |
| **FR-4c** | Cosmic buttons show **sparkle on tap**; off when reduced motion |
| **FR-4d** | Cosmic copy is **Snow Forest Path** (§6); enum mapping unchanged |
| **FR-4e** | Toggle shows **Cosmic** + subtitle *Let the universe choose for me*; `quiz_mode: cosmic` in events |
| **FR-4f** | Serious mode **unchanged** |
| **FR-4g** | Results reveal copy per §8 |
| **FR-4h** | **Floating candle ambience** on Cosmic question screens; static when reduced motion |
| **FR-4i** | Page background **remains light** (`--color-bg`); no full dark stage |
| **FR-4j** | `story_arc_version: "forest-v1c"` on `quiz_started` and `quiz_completed` |

### Acceptance criteria (testable)

- [ ] Cosmic toggle shows subtitle when Cosmic selected (or always visible under Cosmic pill — implementer choice; must be readable)
- [ ] Scenes 1–4: illustration + scene progress + candle + 4 colored buttons on 375px
- [ ] Ninja **Serious** path → Rusutsu/Kiroro/Niseko top 3 (`docs/quiz-scoring.md` §6)
- [ ] Cosmic enum answers → **identical podium** to pre–Epic 8 cosmic mapping
- [ ] `quiz_started` / `quiz_completed` include `quiz_mode: cosmic` + `story_arc_version: forest-v1c`
- [ ] Reduced motion: no sparkle, no candle float/flicker
- [ ] Illustration total ≤500KB; LCP regression ≤200ms vs pre–Epic 8 baseline
- [ ] Illustration attribution documented; Info credit if license requires

---

## 10. Non-goals (Epic 8)

- Scoring weights or 5th question
- Illustrations or candle on **Serious** mode
- **Full dark cosmic stage** (deferred — PO may revisit)
- Sound / haptics
- **Animated scene illustrations** (GIF/video/Lottie for scenes)
- 16 per-answer illustrations
- Renaming `cosmic` in analytics schema
- Thai Cosmic copy in v1 (English only; Serious Thai unchanged)

---

## 11. Success metrics

| ID | Metric | Target |
|----|--------|--------|
| **SM-8a** | Cosmic selection rate | ≥25% of quiz starts (baseline from PostHog after 2 weeks) |
| **SM-8b** | Cosmic completion rate | ≥ Serious completion within 4 weeks |
| **SM-8c** | Quiz → detail tap (Cosmic) | ≥1 podium tap per completed Cosmic quiz (median) |

**Counter-metric:** Do not optimize time-on-quiz if completion drops.

---

## 12. Open questions

1. **Thai locale:** Ship Cosmic copy English-only in Epic 8, or hide Cosmic until translated? *(Default: English Cosmic OK in v1.)*
2. **Subtitle placement:** Under toggle only when Cosmic active vs always visible under Cosmic segment? *(Design spike in 8.4.)*

---

## 13. Assumptions

- AI style lock achieves 4 cohesive scenes within 2 generation rounds.
- Floating candle reads as magical on light bg without cluttering SE viewport.
- Free licensed fallback acceptable if AI QC fails.

---

## 14. See also

| Need | Document |
|------|----------|
| Epic stories | [`../epics/epic-08-quiz-cosmic-story.md`](../epics/epic-08-quiz-cosmic-story.md) |
| Illustration pipeline | [`../../../content/quiz-illustrations/README.md`](../../../content/quiz-illustrations/README.md) |
| Baseline quiz spec | [`../features/quiz.md`](../features/quiz.md) |
| Scoring | [`../../../docs/quiz-scoring.md`](../../../docs/quiz-scoring.md) |
| Motion rules | [`14-ux-components-motion.md`](14-ux-components-motion.md) |

---

## 15. Appendix — AI style lock prompt (v1)

Use the **same style block** for every scene; only change the scene description line.

**Style anchor (append to every generation):**

> Soft editorial watercolor illustration, muted Japanese winter palette, cream and slate blue and pine green, gentle gouache texture, thin consistent ink outlines, calm Kinfolk magazine aesthetic, no anime, no 3D, no photorealism, no people, no text, no logos, horizontal composition, plenty of negative space.

**Scene prompts:**

| File | Scene description |
|------|-------------------|
| `q1.webp` | Snow-covered forest trail splitting into two paths, wooden signpost, paper lantern glow on the gentler left path, dark pine trees, peaceful mood |
| `q2.webp` | Forest opening to wide snowy valley at dusk, soft purple sky, tiny warm village lights in far distance, open powder field foreground |
| `q3.webp` | Three snow-covered Nordic stone arch gates in a row, carved rock pillars, subtle hints: open white field, distant train silhouette, rising onsen steam — not torii |
| `q4.webp` | Old stone clock half-buried in snow, four clock faces or seasonal carvings suggesting winter stages, mystical but minimal, turning hand optional |

**QC checklist:** Same line weight, same sky treatment, same snow texture, no human figures, exports compress to ≤120KB WebP each.
