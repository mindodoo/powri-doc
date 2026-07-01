<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §3: Phase 1 Functional Requirements (FR-1–7)

**Source:** [`../prd-phase1.md`](../prd-phase1.md) §3 · **Detail shards:** `03-*` through `07-*` · **Epics:** [`epics/requirements-inventory.md`](../../../epics/requirements-inventory.md)

Phase 1 FR IDs only. **FR-6 deferred to Phase 3** (monetisation gate). Phase 2+ features (§3.3–3.5, 3.7) are not listed here.

---

| ID | Feature | Phase | Detail shard |
|----|---------|-------|--------------|
| **FR-1** | Resort Discovery (Home & Explore) | 1 | [`03-features-discovery.md`](03-features-discovery.md) |
| **FR-2** | Resort Detail Page | 1 | [`05-features-detail-getting-there.md`](05-features-detail-getting-there.md) |
| **FR-3** | Getting There | 1 | [`05-features-detail-getting-there.md`](05-features-detail-getting-there.md) |
| **FR-4** | Find My Resort Quiz | 1 | [`04-features-quiz-onboarding.md`](04-features-quiz-onboarding.md) · [`features/quiz/brief.md`](../../../features/quiz/brief.md) |
| **FR-5** | Onboarding | 1 | [`04-features-quiz-onboarding.md`](04-features-quiz-onboarding.md) |
| **FR-6** | AI Chat (Resort Q&A) | **3** | [`06-features-ai-chat.md`](06-features-ai-chat.md) · [`epics/phase1/shards/phase-3-monetisation-ai.md`](../../../epics/phase1/shards/phase-3-monetisation-ai.md) |
| **FR-7** | Info | 1 | [`07-features-info.md`](07-features-info.md) |

---

## Acceptance criteria summary

### FR-1 — Resort Discovery
- [ ] All **20** seed resorts as browsable cards (hero, name, region, tagline, badges)
- [ ] Filter chips + search by name; no full page reload
- [ ] Card tap → resort detail (`resort_slug` in analytics)

### FR-2 — Resort Detail
- [ ] Hero, quick verdict, highlights/lowlights, terrain data, season-labelled T2, `trail_map_url` link-out
- [ ] Getting There reachable from detail
- [ ] Unsplash attribution on every hero (§17.2)

### FR-3 — Getting There
- [ ] ≥2 transport options per resort with duration/cost + booking links
- [ ] Gateway city ≥3 experience cards where applicable
- [ ] Practical tips collapsible on mobile

### FR-4 — Quiz
- [ ] 4-question sliding-card quiz from onboarding and Home/Explore
- [ ] Serious / Cosmic mode toggle before Q1 (default Serious); shared scoring enums
- [ ] Completion → **podium top 3** + **results reveal** (Serious: header; Cosmic: interstitial + header + subline) — see `features/quiz/brief.md` § Results podium
- [ ] `quiz_started`, `quiz_completed` with `quiz_mode`, **`result_count: 3`** per tracking plan

### FR-5 — Onboarding
- [ ] ≤4 swipeable cards; skip + "Get started" work
- [ ] Not shown again after completion (local flag)
- [ ] No hero photography on cards

### FR-6 — AI Chat *(Phase 3 — deferred)*
- Moved to Phase 3 with Story 3.6 + Epic 4. Gate: monetisation/sponsorship before high-cost API features.
- See [`06-features-ai-chat.md`](06-features-ai-chat.md) and [`epics/phase1/shards/phase-3-monetisation-ai.md`](../../../epics/phase1/shards/phase-3-monetisation-ai.md) for full ACs when Phase 3 starts.

### FR-7 — Info
- [ ] Info tab: Privacy, Terms (§17.1), about, version, contact/email for data rights
