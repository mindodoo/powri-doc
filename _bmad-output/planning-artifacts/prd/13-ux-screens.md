# PRD §12.5: Screen Specifications

**Source:** [`../prd.md`](../prd.md) §12.5 · **Flows:** [`../ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md`](../ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md) · **Mocks:** `ux-designs/.../mockups/`

---

## Screen 0: Onboarding · FR-5 {#screen-0-onboarding}

- 3–4 full-screen swipeable cards; no photos
- Card 1: app name + tagline + small icon
- Cards 2–4: find resort, plan trip, optional AI
- Progress dots; skip top-right; "Get started" on final
- Final card → quiz entry
- Background `--color-bg`; warm copy tone

---

## Screen 0b: Find My Resort Quiz · FR-4 {#screen-0b-quiz}

- Sliding cards; 4 questions max; full-width answer buttons
- Entry: onboarding, Home, Explore
- **Mode toggle:** Serious (default) or Cosmic before Q1 — see [`../features/quiz.md`](../features/quiz.md)

### Serious mode (canonical)

| # | Question | Options |
|---|----------|---------|
| 1 | What's your level on the mountain? | Just starting out · Getting confident · I charge hard · It varies in my group |
| 2 | What matters most to you? | The powder & terrain · Village & culture · Family-friendly · All of the above |
| 3 | Where does your flight land? | Hokkaido (Sapporo) · Honshu (Tokyo/Nagano) · Tohoku (Sendai) · Not sure yet |
| 4 | When are you thinking of going? | Early season (Dec–Jan) · Peak powder (Feb) · Late season (Mar–Apr) · Still planning |

### Cosmic mode (playful · same scoring)

| # | Question | Options |
|---|----------|---------|
| 1 | You hear the phone ring in the morning. Your friend asks if you want an easy run at the park nearby. You reply: | "Who is this? I'm still asleep." · "Sure — give me twenty minutes. One easy lap." · "I'm already at the park." · "Depends who's asking — it's complicated." |
| 2 | When a day goes really well, the evening looks like… | Replaying the best run… · Lanterns, noodles… · Everyone fed, warm… · A bit of everything… |
| 3 | Pick the winter postcard you'd actually send home. | Hokkaido dairy cow · Shinkansen + soba · Onsen town in the north · Blank card |
| 4 | One travel book won't let you leave. It's called: | *Before the Crowds Arrive* · *The Week Everything Peaks* · *Long Afternoons & Lighter Layers* · *Places I Might Go Eventually* |

Full Cosmic copy + enum mapping: [`../features/quiz.md`](../features/quiz.md)

**Results reveal + podium (required · Story 2.7):**

| Mode | Flow |
|------|------|
| Serious | Header → **podium top 3** (featured #1 + minimal #2/#3) → Browse CTA → Explore |
| Cosmic | Interstitial (~800ms) → header → subline → **podium top 3** → Browse CTA |

- **#1:** accent border, rank **"1"** + label, taller hero, full detail, **match reason**
- **#2/#3:** 48px thumb, name, region only — no tagline/badges
- **No answer summary** tags under header
- Home/Explore/search unchanged — full `ResortCard` lists

Copy and i18n keys: [`../features/quiz.md`](../features/quiz.md) § Results reveal + § Results podium.

---

## Screen 1: Home · FR-1 {#screen-1-home}

- Sticky top bar: logo + search; transparent→white on scroll
- Hero carousel: 65vh, crossfade 3–4s hold / 1s fade, gradient + tagline
- Filter chips below hero
- Resort card list (all 20 reachable ≤2 taps)
- Shared-element transition card→detail hero

---

## Screen 2: Resort Detail · FR-2

- Hero + `QuickVerdict` pull quote
- Highlights/lowlights row
- Stat row, terrain data, trail map link
- Getting There section or link
- Floating "Plan with AI →" above bottom nav

---

## Screen 4: Getting There · FR-3

Transport rows, gateway experiences, practical tips (collapsible).

---

## Screen 5: AI Chat · FR-6

Full-screen overlay from detail. User/AI bubbles; typing indicator.

---

## Screen 6: Info · FR-7

Minimal, no photography. About, legal links, version, contact.

---

## Phase 2 screens (deferred)

Screen 3: Accommodation & Map — do not build Phase 1.

**Full prose specs:** [`../prd.md`](../prd.md) lines ~996–1127
