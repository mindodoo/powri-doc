# Feature Brief: Find My Resort Quiz

**Story:** 2.7 · **FR:** FR-4 · **Epic:** 2 Discover Your Resort · **Phase:** 1  
**Route:** `/quiz` · **Epic 8 (planned):** Cosmic UX v2 → [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../prds/phase1/shards/18-quiz-cosmic-story-ux.md)

> **Start here** for quiz work. You do not need the full PRD or epics. Follow links below only if you need deeper context.  
> **Cosmic improvements (narrative, illustrations, sparkle UI):** see Epic 8 PRD — do **not** implement in 2.7 scope.

---

## Summary

A 4-question, full-screen sliding-card quiz helps undecided users get a personalised resort shortlist. Entry points: onboarding final card (→ Explore with CTA), Home, Explore `FindMyResortCta`. On completion, show a **podium of top 3 resorts** (featured #1 + minimal #2/#3) with mode-specific header copy — **not** the full 20-card list used on Home/Explore. Scoring still ranks all 20 internally; only ranks 1–3 render on the quiz results screen. Scoring is **client-side** in Phase 1.

Users choose **Serious** or **Cosmic** question copy before starting. Both variants map to the **same four scoring enums** — one `score.ts` path, no duplicate algorithms.

---

## User story

> As an undecided planner, I want a short 4-question quiz that recommends resorts, so that I get a personalised starting list without reading all 20.

---

## Acceptance criteria (from epics)

- **Given** entry from onboarding final card, Home, or Explore CTA  
- **When** I start the quiz → `quiz_started` fires with `entry_point` and `quiz_mode`  
- **Then** I can switch **Serious** / **Cosmic** before the first question (default: Serious)  
- **And** four `ResortQuizCard` screens show the active variant's copy with **wide full-width answer buttons** (UX-DR12)  
- Swiping or tapping advances to the next question  
- **Serious results:** header (`quiz.results.serious`) → **podium top 3** (see § Results podium)  
- **Cosmic results:** ~800ms interstitial (`quiz.reveal.cosmic.loading`) → header (`quiz.results.cosmic`) + subline (`quiz.results.cosmicSubline`) → **podium top 3** (see § Results reveal + § Results podium)  
- **No answer summary** tags under the header (e.g. no "Advanced · Powder · Hokkaido · Feb")  
- Secondary CTA **"Browse all resorts →"** links to Explore (full list unchanged there)  
- `quiz_completed` fires after results are shown (not before interstitial), with scoring enums, `quiz_mode`, and **`result_count: 3`**  
- Tapping a podium result → resort detail with `list_context: quiz` and `position` 1 / 2 / 3

---

## Quiz modes

| Mode | `quiz_mode` enum | Purpose |
|------|------------------|---------|
| **Serious** (default) | `serious` | Direct, realistic questions — PRD §12.5 canonical copy |
| **Cosmic** | `cosmic` | **Snow Forest Path** — four-scene illustrated fable; subtitle *Let the universe choose for me* (Epic 8) |

**Toggle UX:** Segmented control on quiz intro (before Q1). **Cosmic subtitle** visible when Cosmic is selected (or under Cosmic segment — see Epic 8.4). Switching mode updates display only; answers reset if toggled mid-quiz (default: toggle only before Q1).

**Scoring rule:** Each answer option maps to a stable enum (see table below). `score.ts` consumes enums only — never display strings.

---

## Questions & answers

### Variant A — Serious (default · PRD §12.5)

| # | Question | Answer options (full-width tappable buttons) |
|---|----------|---------------------------------------------|
| 1 | What's your level on the mountain? | Just starting out · Getting confident · I charge hard · It varies in my group |
| 2 | What matters most to you? | The powder & terrain · Village & culture · Family-friendly · All of the above |
| 3 | Where does your flight land? | Hokkaido (Sapporo) · Honshu (Tokyo/Nagano) · Tohoku (Sendai) · Not sure yet |
| 4 | When are you thinking of going? | Early season (Dec–Jan) · Peak powder (Feb) · Late season (Mar–Apr) · Still planning |

### Variant B — Cosmic (Snow Forest Path · Epic 8)

> **Phase 1 shipped** disconnected vignettes (phone call, postcard, etc.). **Epic 8 replaces copy** with one storyline — full text in [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../prds/phase1/shards/18-quiz-cosmic-story-ux.md) §6. Enum keys unchanged.

| # | Scene | Question (summary) |
|---|-------|-------------------|
| 1 | Fork in the trail | Choose your path (skill / level) |
| 2 | Valley at dusk | What makes tonight perfect (priority) |
| 3 | Spirit gates | Which gate pulls you through (region) |
| 4 | Stone winter clock | When your journey begins (timing) |

**Epic 8 UX (Cosmic only):** scene illustration, floating candle, twilight answer buttons + sparkle. Light page background. Serious unchanged.

#### Snow Forest Path — final copy (`forest-v1c` · PO approved)

Full text in [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../prds/phase1/shards/18-quiz-cosmic-story-ux.md) §6. Mixed A/B selections + Scene 4 question update.

**Scene 1** — *You reach a fork in the snow-laced trail. A wooden sign reads: “Choose your path.”*

- You take the lantern-lit path — slow and easy.
- The worn path looks safe; you'll try it and see how it goes.
- A narrow opening in the pines — only you noticed it — and you slip through.
- You wait at the sign for the others — you'll follow their lead.

**Scene 2** — *The forest opens to a valley. What would make tonight perfect?*

- You stay out in the quiet snow — just you and the mountains.
- You walk toward the village lights, food, and narrow streets.
- Smoke rises from a hut where voices sound safe — you head toward the fire.
- Why choose one ending? You'll take the long way and have a little of everything.

**Scene 3** — *Three gates stand in the snow. Which one pulls you through?*

- The far gate — wide open snow and cold northern wind.
- The middle gate hums like distant rails; steam curls up from a bowl you can't see yet.
- The northern gate breathes hot spring mist against the cold air.
- You don't walk through any gate — you're not ready to choose.

**Scene 4** — *You find a stone clock in the snow. Four faces mark different depths of winter. When does your journey begin?*

- The face of thin light — early footsteps, an empty world still waking.
- The face locked in deepest winter — the hour when everything holds still.
- The face where snow softens and the shadows grow long.
- The hand keeps turning — you'll know when it finally stops.

#### Legacy Cosmic copy (pre–Epic 8 — replace on ship)

| # | Question |
|---|----------|
| 1 | Phone call / park nearby |
| 2 | When a day goes really well… |
| 3 | Winter postcard |
| 4 | Bookstore travel book |

---

## Scoring enum mapping (both variants)

| Axis | Enum key | Values | Serious → enum | Cosmic → enum |
|------|----------|--------|----------------|---------------|
| Q1 Skill / intensity | `level` | `beginner` · `confident` · `advanced` · `group` | (Serious labels) | lantern glow, unhurried → `beginner` · well-walked trail, test once → `confident` · hidden gap in pines → `advanced` · wait for footsteps → `group` |
| Q2 Trip priority | `priority` | `powder_terrain` · `village_culture` · `family` · `all` | (Serious labels) | forest quiet, day in your bones → `powder_terrain` · lantern roofs, broth-steam → `village_culture` · hut fire, safe voices → `family` · long way, a little of everything → `all` |
| Q3 Gateway / region | `region` | `hokkaido` · `honshu` · `tohoku` · `unsure` | (Serious labels) | white silence, cowbell → `hokkaido` · distant rails, unseen bowl → `honshu` · hot spring mist → `tohoku` · wait in the grey → `unsure` |
| Q4 Season / timing | `timing` | `early_season` · `peak_february` · `late_season` · `still_planning` | (Serious labels) | thin light, waking world → `early_season` · deepest winter, holds still → `peak_february` · softening snow, long shadows → `late_season` · hand still turning → `still_planning` |

Implement in `web/src/lib/quiz/questions.ts` as `{ id, enumValue }` per option; i18n keys like `quiz.serious.q1.options.beginner` and `quiz.cosmic.q1.options.beginner`.

---

## Results reveal *(required · Story 2.7)*

### Serious mode

| Element | Copy | i18n key |
|---------|------|----------|
| Header | Based on your answers, here are resorts we'd recommend for you. | `quiz.results.serious` |

No interstitial. Podium appears immediately after Q4 (200ms slide transition).

### Cosmic mode *(required · Option A — Oracle)*

After the last answer, before the resort list:

| Step | Element | Copy | i18n key | Required |
|------|---------|------|----------|----------|
| 1 | **Interstitial** | *Consulting the powder oracle…* | `quiz.reveal.cosmic.loading` | ✅ ~800ms crossfade |
| 2 | **Header** | *The stars have spoken. Here are your resorts:* | `quiz.results.cosmic` | ✅ |
| 3 | **Subline** | *Ranked by cosmic vibes and extremely real resort data.* | `quiz.results.cosmicSubline` | ✅ smaller/caption typography |
| 4 | **Podium** | Top 3 only — featured #1 + minimal #2/#3 (§ Results podium) | — | ✅ slides up after step 1–3 |
| 5 | **Browse CTA** | *"Browse all resorts →"* links to Explore | `quiz.results.browseAll` | ✅ below podium |

**Motion:** Interstitial crossfade ~800ms; podium enter 200ms ease-out (PRD §12.8). No bounce.  
**Analytics:** Fire `quiz_completed` after interstitial completes and results header is visible (not mid-interstitial). `result_count` = **3**.

### Copy alternatives (not Phase 1 — do not implement)

| Style | Interstitial | Header |
|-------|--------------|--------|
| B — Playful honest | Crunching the vibes… | We asked the universe (and ran the numbers). Start here: |
| C — Minimal | *(none)* | Your cosmic ski reading — ranked most aligned first: |
| D — Two-line editorial | *(none)* | **Your mountain match** + subline |

**Deferred (Phase 2):** Priority-keyed one-liner readings (e.g. powder → *"The flakes are calling."*).

Podium layout and scoring are identical for both modes; only reveal copy differs.

---

## Results podium *(required · Story 2.7)*

Quiz results show **top 3 ranked resorts only**. `score.ts` still ranks all 20; ranks 4–20 are not displayed on this screen. Home, Explore, and search are **unchanged** — full `ResortCard` lists.

### Layout

```
[Reveal header — Serious or Cosmic]
[#1 QuizTopMatchCard — featured]
[#2 QuizRunnerUpRow — minimal]
[#3 QuizRunnerUpRow — minimal]
[Browse all resorts → Explore]
```

### #1 — Featured top match (`QuizTopMatchCard`)

| Aspect | Spec |
|--------|------|
| **Highlight** | 2px `--color-accent` border; accent tint on rank column (see `key-quiz-results.html` mock) |
| **Labels** | **Both:** serif rank **"1"** pill **and** text — Serious: *"Your top match"* (`quiz.results.topMatchLabel`) · Cosmic: *"Your cosmic #1"* (`quiz.results.topMatchLabelCosmic`) |
| **Hero** | Taller than standard card — `aspect-[16/9]` (Explore/Home use `16/10`) |
| **Content** | Full detail: hero image, region pill, name, tagline, 2–3 badge pills, chevron |
| **Match reason** | One accent line below name/region — e.g. *"Strong match — powder & terrain"* — from `getMatchReason()` (see [`features/quiz/scoring.md`](scoring.md) §4.1) |
| **Tap** | Whole card → detail; `resort_card_tapped` with `position: 1`, `list_context: quiz` |

### #2 & #3 — Minimal runner-ups (`QuizRunnerUpRow`)

| Aspect | Spec |
|--------|------|
| **Layout** | Compact horizontal row; ~56–64px tap target (≥44px a11y) |
| **Thumbnail** | **48×48** rounded image |
| **Content** | Rank badge **2** / **3** · resort name (serif) · region (uppercase caption) · chevron |
| **Excluded** | No large hero, tagline, badge pills, or match reason |
| **Tap** | Row → detail; `position: 2` / `position: 3` |

### Browse CTA

- Text: *"Browse all resorts →"* (`quiz.results.browseAll`)
- Navigates to Explore tab — user gets the normal full resort list + filters there
- Placed below the podium with comfortable spacing

---

## UX notes

- Sliding card quiz — not a form. Each question = one full-screen card; next slides in from the right (200ms, see EXPERIENCE.md)
- Forward-only assumed; skip quiz control in mock (`key-quiz.html`)
- **Serious:** no photographs on quiz cards (fast load)
- **Cosmic (Epic 8):** one scene illustration per question; floating candle SVG; colored sparkle buttons on **light** page background
- Cosmic Q4 uses **stone winter clock** scene (replaces sundial / book-title patterns)

---

## Components & design

| Item | Spec |
|------|------|
| `QuizModeToggle` | Serious / Cosmic; Cosmic subtitle (Epic 8) |
| `ResortQuizCard` | Full-screen; `variant="cosmic"` for illustration + candle + colored buttons |
| `CosmicCandleAmbience` | Floating lantern SVG (Epic 8) |
| `QuizResultsReveal` | Serious: header only. Cosmic: interstitial → header + subline → podium (§ Results reveal) |
| `QuizTopMatchCard` | Featured #1 — accent border, dual labels, taller hero, match reason, full card detail |
| `QuizRunnerUpRow` | Minimal #2/#3 — 48px thumb, name, region, rank badge; no tagline/badges |
| `getMatchReason()` | `web/src/lib/quiz/matchReason.ts` — strongest axis → i18n one-liner (§4.1 in scoring doc) |
| Browse CTA | Link to Explore below podium — does not affect Explore list behaviour |
| Motion | Quiz advance + list enter: 200ms ease-out; Cosmic interstitial: ~800ms crossfade; no bounce (PRD §12.8) |

**Mocks:**  
- [`../ux-designs/ux-phase1/mockups/key-quiz.html`](../../ux-designs/ux-phase1/mockups/key-quiz.html)  
- [`../ux-designs/ux-phase1/mockups/key-quiz-results.html`](../../ux-designs/ux-phase1/mockups/key-quiz-results.html)

---

## Architecture

From [`architecture/phase1/architecture-phase1.md`](../../architecture/phase1/architecture-phase1.md):

| Concern | Decision |
|---------|----------|
| State | Zustand `useQuizStore` — `quizMode`, answers (enum values), client session only |
| Scoring | `web/src/lib/quiz/score.ts` — **enum values only** → resort filter predicates |
| Questions | `web/src/lib/quiz/questions.ts` — `{ serious, cosmic }` × 4 questions × 4 options with shared `enumValue` |
| i18n | `web/messages/en.json` — `quiz.serious.*`, `quiz.cosmic.*`, `quiz.results.*`, `quiz.reveal.cosmic.loading`, `quiz.results.topMatchLabel`, `quiz.results.topMatchLabelCosmic`, `quiz.results.browseAll`, `quiz.results.matchReason.*` |
| Components | `web/src/components/quiz/` — `QuizModeToggle`, `ResortQuizCard`, `QuizResultsReveal`, `QuizTopMatchCard`, `QuizRunnerUpRow` |
| Page | `web/src/app/[locale]/quiz/page.tsx` (stub today) |
| Data | Same build-time resort index as Home/Explore — **no separate data source** |

**Scoring:** Spec in [`features/quiz/scoring.md`](scoring.md) — implement `web/src/lib/quiz/score.ts` against §3. Serious and Cosmic share one algorithm. Validate with Ninja persona (§6) before sign-off.

---

## Analytics (`docs/analytics/tracking-plan.md`)

| Event | When | Key properties |
|-------|------|----------------|
| `quiz_started` | Quiz begins | `entry_point`, **`quiz_mode`** (`serious` / `cosmic`) |
| `quiz_completed` | Results reveal visible (Cosmic: **after** ~800ms interstitial) | **`level`**, **`priority`**, **`region`**, **`timing`** (enum values), **`quiz_mode`**, **`result_count: 3`** |
| `resort_card_tapped` | Result card tap | `list_context: quiz`, `resort_slug`, `position` |

Update tracking-plan in the same change as instrumentation. Verify in PostHog before sign-off.

---

## Persona validation (EXPERIENCE.md + `features/quiz/scoring.md` §6)

| Persona | Quiz path | Pass highlight |
|---------|-----------|----------------|
| **Ninja** (Flow 2) | advanced · powder · Hokkaido · Feb | Rusutsu/Kiroro/Niseko in top 3 |
| **Alex** (Flow 3) | confident · family · Hokkaido · still planning | Furano/Sahoro/Tomamu in top 5 |
| **Mind** | confident · all · unsure · peak Feb | Nozawa + Hakuba top 3; Madarao top 5 |

Ninja Serious: *I charge hard* → *Powder & terrain* → *Hokkaido* → *Peak powder (Feb)*.  
Mind Serious: *Getting confident* → *All of the above* → *Not sure yet* → *Peak powder (Feb)*.

**Success (Ninja):** Top 3 include at least one resort he respects; **#1 visually dominant** (featured card + match reason); ranking defensible.  
**Success (Mind):** Nozawa/Madarao/Hakuba cluster in **visible top 3**; #1 clearly highlighted vs #2/#3 minimal rows.

**Failure:** Results feel random → user ignores quiz and uses Explore filters instead.

---

## Existing code & dependencies

| Artifact | Status |
|----------|--------|
| Story 2.6 Explore + `FindMyResortCta` → `/quiz` | ✅ Done |
| `/quiz` stub page | ✅ Placeholder copy only |
| Filter/list components | ✅ Reuse from Epic 2 |
| Onboarding → `/explore` (not direct `/quiz`) | ✅ Acceptable; CTA visible on Explore |
| `features/quiz/scoring.md` | ✅ Weights + persona fixtures — implement `score.ts` against §3 |

**Blocked by:** None (2.6 complete). **Implement against:** [`features/quiz/scoring.md`](scoring.md).

---

## Deferred (do not build in 2.7)

- Persist quiz preferences or preferred mode to user account (Phase 3)
- Quiz back-navigation UX (confirm if needed; forward-only assumed)
- Automated unit tests for scoring (optional Phase 1; recommended if logic grows)
- A/B default mode by entry point (future experiment)

---

## See also (read only if needed)

| Need | Document |
|------|----------|
| FR acceptance checklist | [`prds/phase1/shards/04-features-quiz-onboarding.md`](../../prds/phase1/shards/04-features-quiz-onboarding.md) |
| Full story + epic context | [`epics/phase1/shards/epic-02-discover.md`](../../epics/phase1/shards/epic-02-discover.md#story-27) |
| Screen 0b full spec | [`prds/phase1/shards/13-ux-screens.md`](../../prds/phase1/shards/13-ux-screens.md) |
| Component visual tokens | [`ux-designs/ux-phase1/design.md`](../../ux-designs/ux-phase1/design.md) |
| User flows | [`ux-designs/ux-phase1/experience.md`](../../ux-designs/ux-phase1/experience.md) |
| Resort scoring algorithm | [`features/quiz/scoring.md`](scoring.md) |
| Resort content schema | [`content/content-model.md`](../../../../content/content-model.md) |
| Full PRD §12.5 | [`prds/phase1/prd-phase1.md`](../../prds/phase1/prd-phase1.md) Screen 0b |
| Epic 8 Cosmic UX | [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](../../prds/phase1/shards/18-quiz-cosmic-story-ux.md) |
| Illustration assets | [`content/quiz-illustrations/README.md`](../../../../content/quiz-illustrations/README.md) |
| Epic 8 QA | [`docs/qa/phase1/manual-qa-epic-08.md`](../../../../docs/qa/phase1/manual-qa-epic-08.md) |
