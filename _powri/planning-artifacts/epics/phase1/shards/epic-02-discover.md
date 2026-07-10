<!-- generated from epics/phase1/epics-phase1.md — do not edit; run prd-shard-regen.md -->

# Epic 2: Discover Your Resort

**Stories:** 2.1–2.7 (+ 2.2.1, 2.7.1) · **Status:** Complete (see retro) · **FRs:** FR-1, FR-4, FR-5 · **PRD:** [`prds/phase1/shards/03-features-discovery.md`](../../../prds/phase1/shards/03-features-discovery.md), [`prds/phase1/shards/04-features-quiz-onboarding.md`](../../../prds/phase1/shards/04-features-quiz-onboarding.md)

Onboarding, Home hero, resort cards, filters, search, Explore tab, Find My Resort quiz.

---

## Story 2.1: First-Time Onboarding Flow ✅

≤4 `OnboardingCard` slides; no photos; skip + "Get started"; persistent flag; final card → quiz entry.

**Artifact:** [`_powri/implementation-artifacts/Epic-2/2-1-first-time-onboarding-flow.md`](../../../../implementation-artifacts/Epic-2/2-1-first-time-onboarding-flow.md)

---

## Story 2.2: Home Hero Carousel & Tagline ✅

`HeroCarousel` crossfade from `content/site-images.md`; tagline; Unsplash attribution; sticky top bar scroll behaviour.

**Artifacts:** 2-2, 2-2-1 (compact hero height)

---

## Story 2.3: Resort List & Resort Cards (All 20) ✅

`ResortCard` for all 20; shared-element transition; `resort_card_tapped` with `list_context: home`.

**Artifact:** [`_powri/implementation-artifacts/Epic-2/2-3-resort-list-resort-cards-all-20.md`](../../../../implementation-artifacts/Epic-2/2-3-resort-list-resort-cards-all-20.md)

---

## Story 2.4: Filter Chips & Instant List Filtering ✅

Client-side filter; single-select chips; `filter_applied` / `filter_zero_results`.

**Artifact:** [`_powri/implementation-artifacts/Epic-2/2-4-filter-chips-instant-list-filtering.md`](../../../../implementation-artifacts/Epic-2/2-4-filter-chips-instant-list-filtering.md)

---

## Story 2.5: Inline Search by Resort Name ✅

Inline search from top bar; live results; `search_performed`.

**Artifact:** [`_powri/implementation-artifacts/Epic-2/2-5-inline-search-by-resort-name.md`](../../../../implementation-artifacts/Epic-2/2-5-inline-search-by-resort-name.md)

---

## Story 2.6: Explore Tab ✅

Filters + quiz CTA prominent; same dataset as Home; `list_context: explore` / `filtered`.

**Artifact:** [`_powri/implementation-artifacts/Epic-2/2-6-explore-tab.md`](../../../../implementation-artifacts/Epic-2/2-6-explore-tab.md)

---

## Story 2.7: Find My Resort Quiz {#story-27}

**→ Feature brief:** [`features/quiz/brief.md`](../../../features/quiz/brief.md) ← **start here**

As an undecided planner, I want a short 4-question quiz that recommends resorts, so that I get a personalised starting list without reading all 20.

**Acceptance Criteria:**

- **Given** entry from onboarding final card, Home, or Explore CTA
- **When** I start the quiz → `quiz_started` with `entry_point` and `quiz_mode` (`serious` default)
- **Then** I can toggle **Serious / Cosmic** before Q1; four `ResortQuizCard` screens with wide buttons (FR-4, UX-DR12)
- Each answer maps to shared enums (`level`, `priority`, `region`, `timing`) — one `score.ts` for both modes
- Swipe or tap advances
- **Results reveal:** Serious → header only; Cosmic → interstitial (~800ms) + header + subline → **podium top 3** ([`features/quiz/brief.md`](../../../features/quiz/brief.md) § Results reveal + § Results podium)
- **Podium:** #1 featured (`QuizTopMatchCard` — accent border, dual labels, match reason) + #2/#3 minimal (`QuizRunnerUpRow` — 48px thumb); no answer summary tags
- **Browse CTA:** *"Browse all resorts →"* → Explore; `score.ts` ranks all 20 internally
- `quiz_completed` after reveal visible (Cosmic: after interstitial)
- `quiz_completed` with enums, `quiz_mode`, **`result_count: 3`**
- Podium tap → detail with `list_context: quiz`, `position` 1/2/3

**Question copy:** Serious (PRD §12.5) + Cosmic (playful) — full tables in feature brief.

**Prerequisites:** [`features/quiz/scoring.md`](../../../features/quiz/scoring.md) defines weights. `/quiz` stub exists from 2.6.

---

**Full story text:** [`epics/phase1/epics-phase1.md`](../epics-phase1.md) Story 2.7

---

**Implementation artifacts:** [`_powri/implementation-artifacts/Epic-2/`](../../../../implementation-artifacts/Epic-2/)  
**Retro:** [`_powri/implementation-artifacts/Epic-Retro/epic-2-retro-2026-06-13.md`](../../../../implementation-artifacts/Epic-Retro/epic-2-retro-2026-06-13.md)
