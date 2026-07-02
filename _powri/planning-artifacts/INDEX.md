# Planning Artifacts — AI Agent Index

**Project:** Powri — Japan ski trip planner (`_powri/` planning artifacts)  
**Phase 1 scope:** Home · Explore · Info · 20 resorts · quiz · **no AI** · no Plan tab

Use this file to decide **what to read** before coding.

---

## Source-of-truth rules

| Layer | Phase 1 | Phase 2 |
|-------|---------|---------|
| **PRD monolith** | [`prds/phase1/prd-phase1.md`](prds/phase1/prd-phase1.md) | [`prds/phase2/prd-phase2.md`](prds/phase2/prd-phase2.md) |
| **PRD shards** | [`prds/phase1/shards/`](prds/phase1/shards/) — read-only | [`prds/phase2/shards/`](prds/phase2/shards/) — read-only |
| **Epic monolith** | [`epics/phase1/epics-phase1.md`](epics/phase1/epics-phase1.md) | [`epics/phase2/epics-phase2.md`](epics/phase2/epics-phase2.md) |
| **Epic shards** | [`epics/phase1/shards/`](epics/phase1/shards/) — read-only | [`epics/phase2/shards/`](epics/phase2/shards/) — read-only |

1. **Start here** — task lookup table below.  
2. **Read shards** for your feature (~50–200 lines each).  
3. **Edit monoliths only** — then follow [`prd-shard-regen.md`](prd-shard-regen.md).  
4. **Historical input:** [`input_docs/japan_ski_prd.md`](../../input_docs/japan_ski_prd.md) — do not use for implementation.  
5. **Bootstrap archive:** [`prds/phase1/archive/`](prds/phase1/archive/README.md) — point-in-time only.

---

## Task → document lookup

Epic # matches the `epic_num` in sprint-status.yaml keys (e.g. `9-5-...` → Epic 9) — use it for a direct lookup instead of scanning by feature name.

| Epic # | I'm working on… | Read first (in order) | ~Lines |
|--------|-----------------|----------------------|--------|
| 2 | **Find My Resort quiz** (Story 2.7) | [`features/quiz/brief.md`](features/quiz/brief.md) → [`epics/phase1/shards/epic-02-discover.md`](epics/phase1/shards/epic-02-discover.md#story-27) → [`prds/phase1/shards/04-features-quiz-onboarding.md`](prds/phase1/shards/04-features-quiz-onboarding.md) | ~250 |
| 2 | Onboarding (Story 2.1) | [`epics/phase1/shards/epic-02-discover.md`](epics/phase1/shards/epic-02-discover.md#story-21) → [`prds/phase1/shards/04-features-quiz-onboarding.md`](prds/phase1/shards/04-features-quiz-onboarding.md) → [`prds/phase1/shards/13-ux-screens.md`](prds/phase1/shards/13-ux-screens.md) | ~200 |
| 2 | Home / hero / resort cards (2.2–2.3) | [`epics/phase1/shards/epic-02-discover.md`](epics/phase1/shards/epic-02-discover.md) → [`prds/phase1/shards/03-features-discovery.md`](prds/phase1/shards/03-features-discovery.md) | ~200 |
| 2 | Filters / search / Explore (2.4–2.6) | [`epics/phase1/shards/epic-02-discover.md`](epics/phase1/shards/epic-02-discover.md) → [`prds/phase1/shards/03-features-discovery.md`](prds/phase1/shards/03-features-discovery.md) | ~200 |
| 3 | Resort detail (Epic 3) | [`epics/phase1/shards/epic-03-detail.md`](epics/phase1/shards/epic-03-detail.md) → [`prds/phase1/shards/05-features-detail-getting-there.md`](prds/phase1/shards/05-features-detail-getting-there.md) | ~250 |
| 9–15 | **Phase 2 (auth, UGC, maps, trips)** | [`prds/phase2/shards/03-phase2-functional-requirements.md`](prds/phase2/shards/03-phase2-functional-requirements.md) → [`epics/phase2/epics-phase2.md`](epics/phase2/epics-phase2.md) → [`ux-designs/ux-phase2/experience.md`](ux-designs/ux-phase2/experience.md) | ~150 |
| 9 | **Phase 2 auth (Epic 9)** | [`epics/phase2/shards/epic-09-supabase-auth.md`](epics/phase2/shards/epic-09-supabase-auth.md) → [`prds/phase2/shards/04-features-auth.md`](prds/phase2/shards/04-features-auth.md) → [`architecture/phase2/architecture-phase2.md`](architecture/phase2/architecture-phase2.md) §3–4 | ~200 |
| 10 | **Phase 2 nav & saved (Epic 10)** | [`epics/phase2/shards/epic-10-nav-saved.md`](epics/phase2/shards/epic-10-nav-saved.md) → [`prds/phase2/shards/11-features-saved-resorts.md`](prds/phase2/shards/11-features-saved-resorts.md) · [`12-features-navigation.md`](prds/phase2/shards/12-features-navigation.md) | ~180 |
| 11 | **Phase 2 map (Epic 11)** | [`epics/phase2/shards/epic-11-resort-map.md`](epics/phase2/shards/epic-11-resort-map.md) → [`prds/phase2/shards/05-features-resort-map.md`](prds/phase2/shards/05-features-resort-map.md) → `content-model.md` lat/lng | ~140 |
| 12 | **Phase 2 reviews (Epic 12)** | [`epics/phase2/shards/epic-12-community-reviews.md`](epics/phase2/shards/epic-12-community-reviews.md) → [`prds/phase2/shards/06-features-reviews-comments.md`](prds/phase2/shards/06-features-reviews-comments.md) | ~180 |
| 13 | **Phase 2 passport & profile (Epic 13)** | [`epics/phase2/shards/epic-13-passport-profile.md`](epics/phase2/shards/epic-13-passport-profile.md) → [`prds/phase2/shards/07-features-ski-passport.md`](prds/phase2/shards/07-features-ski-passport.md) · [`13-features-public-profile.md`](prds/phase2/shards/13-features-public-profile.md) | ~200 |
| 14 | **Phase 2 trips (Epic 14)** | [`epics/phase2/shards/epic-14-trip-planning.md`](epics/phase2/shards/epic-14-trip-planning.md) → [`prds/phase2/shards/08-features-trip-skeleton.md`](prds/phase2/shards/08-features-trip-skeleton.md) | ~140 |
| 15 | **Phase 2 launch (Epic 15)** | [`epics/phase2/shards/epic-15-phase2-launch.md`](epics/phase2/shards/epic-15-phase2-launch.md) → [`prds/phase2/shards/09-features-ai-stubs.md`](prds/phase2/shards/09-features-ai-stubs.md) · [`10-features-seo.md`](prds/phase2/shards/10-features-seo.md) · [`16-metrics-analytics-decisions.md`](prds/phase2/shards/16-metrics-analytics-decisions.md) | ~200 |
| — | Review AI theme summaries (Phase 3) | [`epics/phase1/shards/phase-3-user-community.md`](epics/phase1/shards/phase-3-user-community.md) | ~80 |
| 4 | AI chat (Phase 3) | [`epics/phase1/shards/phase-3-monetisation-ai.md`](epics/phase1/shards/phase-3-monetisation-ai.md) → [`epics/phase1/shards/epic-04-ai-assistant.md`](epics/phase1/shards/epic-04-ai-assistant.md) → [`prds/phase1/shards/06-features-ai-chat.md`](prds/phase1/shards/06-features-ai-chat.md) | ~300 |
| 5 | Info / legal / launch (Epic 5) | [`epics/phase1/shards/epic-05-launch.md`](epics/phase1/shards/epic-05-launch.md) → [`prds/phase1/shards/07-features-info.md`](prds/phase1/shards/07-features-info.md) | ~150 |
| 6 | **Phase 1 improvements (Epic 6)** | [`epics/phase1/shards/epic-06-phase1-improvements.md`](epics/phase1/shards/epic-06-phase1-improvements.md) → [`prds/phase1/shards/16-phase1-improvements.md`](prds/phase1/shards/16-phase1-improvements.md) | ~200 |
| 7 | **Phase 1 UX wave 2 (Epic 7)** | [`epics/phase1/shards/epic-07-phase1-ux-wave2.md`](epics/phase1/shards/epic-07-phase1-ux-wave2.md) → [`prds/phase1/shards/17-phase1-improvements-wave2.md`](prds/phase1/shards/17-phase1-improvements-wave2.md) | ~250 |
| 8 | **Cosmic quiz UX v2 (Epic 8)** | [`epics/phase1/shards/epic-08-quiz-cosmic-story.md`](epics/phase1/shards/epic-08-quiz-cosmic-story.md) → [`prds/phase1/shards/18-quiz-cosmic-story-ux.md`](prds/phase1/shards/18-quiz-cosmic-story-ux.md) | ~500 |
| 1 | App foundation (Epic 1) | [`epics/phase1/shards/epic-01-foundation.md`](epics/phase1/shards/epic-01-foundation.md) → [`architecture/phase1/architecture-phase1.md`](architecture/phase1/architecture-phase1.md) | ~200 |
| — | Design tokens / components | [`ux-designs/ux-phase1/design.md`](ux-designs/ux-phase1/design.md) · Phase 2: [`ux-designs/ux-phase2/design.md`](ux-designs/ux-phase2/design.md) | varies |
| — | User flows / personas | [`ux-designs/ux-phase1/experience.md`](ux-designs/ux-phase1/experience.md) · Phase 2: [`ux-designs/ux-phase2/experience.md`](ux-designs/ux-phase2/experience.md) | ~350 |
| — | Analytics events | [`../../docs/analytics/tracking-plan.md`](../../docs/analytics/tracking-plan.md) → [`prds/phase1/shards/15-analytics-legal-glossary.md`](prds/phase1/shards/15-analytics-legal-glossary.md) | |
| — | Quiz scoring | [`features/quiz/scoring.md`](features/quiz/scoring.md) | ~100 |
| — | Resort content | [`../../content/content-model.md`](../../content/content-model.md) → [`prds/phase1/shards/10-content-and-decisions.md`](prds/phase1/shards/10-content-and-decisions.md) | varies |
| — | Sprint / stories | [`../implementation-artifacts/sprint-status.yaml`](../implementation-artifacts/sprint-status.yaml) + story file | varies |

---

## Document map

### Phase 1 PRD

| Monolith | [`prds/phase1/prd-phase1.md`](prds/phase1/prd-phase1.md) |
| Shards index | [`prds/phase1/shards/README.md`](prds/phase1/shards/README.md) |
| Post-launch amendments in monolith | §3.8 Epic 6 · §3.9 Epic 7 · §3.10 Epic 8 |

### Phase 2 PRD

| Monolith | [`prds/phase2/prd-phase2.md`](prds/phase2/prd-phase2.md) |
| Shards index | [`prds/phase2/shards/README.md`](prds/phase2/shards/README.md) |
| FR checklist | [`prds/phase2/shards/03-phase2-functional-requirements.md`](prds/phase2/shards/03-phase2-functional-requirements.md) |
| Addendum | [`prds/phase2/addendum.md`](prds/phase2/addendum.md) |
| Pre-implementation gate | [`prds/phase2/readiness-checklist.md`](prds/phase2/readiness-checklist.md) |

### Epics

| Index | [`epics/README.md`](epics/README.md) |
| Phase 1 monolith | [`epics/phase1/epics-phase1.md`](epics/phase1/epics-phase1.md) |
| Phase 2 monolith | [`epics/phase2/epics-phase2.md`](epics/phase2/epics-phase2.md) |
| FR inventory | [`epics/requirements-inventory.md`](epics/requirements-inventory.md) |

### Other authoritative docs

| Doc | When to read |
|-----|--------------|
| [`architecture/phase1/architecture-phase1.md`](architecture/phase1/architecture-phase1.md) | Phase 1 stack |
| [`architecture/phase2/architecture-phase2.md`](architecture/phase2/architecture-phase2.md) | Phase 2 Supabase, API, env |
| [`architecture/README.md`](architecture/README.md) | Architecture index |
| [`../../docs/analytics/tracking-plan.md`](../../docs/analytics/tracking-plan.md) | Analytics events |
| [`../../content/content-model.md`](../../content/content-model.md) | Resort markdown schema |
| [`../../web/AGENTS.md`](../../web/AGENTS.md) | Next.js conventions |
| [`../../docs/quality/tech-debt-backlog.md`](../../docs/quality/tech-debt-backlog.md) | Open tech debt |

---

*Planning index · Phase 1 launched 2026-06-17 · Phase 2 in progress*
