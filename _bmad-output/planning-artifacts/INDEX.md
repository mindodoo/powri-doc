# Planning Artifacts — AI Agent Index

**Project:** Japan Ski & Snowboard Trip Planner (`japan_winter_sport`)  
**Phase 1 scope:** Home · Explore · Info · 20 resorts · quiz · **no AI** · no Plan tab

Use this file to decide **what to read** before coding. You do **not** need the full 1,400-line PRD or 650-line epics doc for most tasks.

---

## How to use this system

1. **Start here** — pick your task from the lookup table below.
2. **Read the listed shards only** — each shard is ~50–200 lines, cross-linked.
3. **Follow "See also" links** when you hit a dependency (analytics, UX tokens, content schema).
4. **Full sources** (`prd.md`, `epics.md`) remain authoritative for edge cases; shards point to exact PRD sections (§).

**Rule:** Prefer shard + feature brief over loading monoliths. Load full PRD/epics only when a shard says "details in §X" and you need them.

---

## Task → document lookup

| I'm working on… | Read first (in order) | ~Lines |
|-----------------|----------------------|--------|
| **Find My Resort quiz** (Story 2.7) | [`features/quiz.md`](features/quiz.md) → [`epics/epic-02-discover.md`](epics/epic-02-discover.md#story-27) → [`prd/04-features-quiz-onboarding.md`](prd/04-features-quiz-onboarding.md) | ~250 |
| Onboarding (Story 2.1) | [`epics/epic-02-discover.md`](epics/epic-02-discover.md#story-21) → [`prd/04-features-quiz-onboarding.md`](prd/04-features-quiz-onboarding.md) → UX Screen 0 in [`prd/13-ux-screens.md`](prd/13-ux-screens.md) | ~200 |
| Home / hero / resort cards (2.2–2.3) | [`epics/epic-02-discover.md`](epics/epic-02-discover.md) → [`prd/03-features-discovery.md`](prd/03-features-discovery.md) | ~200 |
| Filters / search / Explore (2.4–2.6) | [`epics/epic-02-discover.md`](epics/epic-02-discover.md) → [`prd/03-features-discovery.md`](prd/03-features-discovery.md) | ~200 |
| Resort detail (Epic 3) | [`epics/epic-03-detail.md`](epics/epic-03-detail.md) → [`prd/05-features-detail-getting-there.md`](prd/05-features-detail-getting-there.md) | ~250 |
| **Phase 2 (auth, UGC, maps, trips)** | [`epics/epics-phase2.md`](epics/epics-phase2.md) → [`ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md`](ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md) → [`architecture-phase2.md`](architecture-phase2.md) | ~900 |
| **Phase 2 auth (Epic 9)** | [`epics/epic-09-supabase-auth.md`](epics/epic-09-supabase-auth.md) → [`architecture-phase2.md`](architecture-phase2.md) §3–4 | ~150 |
| **Phase 2 nav & saved (Epic 10)** | [`epics/epic-10-nav-saved.md`](epics/epic-10-nav-saved.md) | ~100 |
| **Phase 2 map (Epic 11)** | [`epics/epic-11-resort-map.md`](epics/epic-11-resort-map.md) → `CONTENT_MODEL.md` lat/lng | ~100 |
| **Phase 2 reviews (Epic 12)** | [`epics/epic-12-community-reviews.md`](epics/epic-12-community-reviews.md) | ~120 |
| **Phase 2 passport & profile (Epic 13)** | [`epics/epic-13-passport-profile.md`](epics/epic-13-passport-profile.md) | ~120 |
| **Phase 2 trips (Epic 14)** | [`epics/epic-14-trip-planning.md`](epics/epic-14-trip-planning.md) | ~80 |
| **Phase 2 launch (Epic 15)** | [`epics/epic-15-phase2-launch.md`](epics/epic-15-phase2-launch.md) → `docs/tracking-plan.md` | ~100 |
| Review AI theme summaries (Phase 3 — deferred) | [`epics/phase-3-user-community.md`](epics/phase-3-user-community.md) | ~80 |
| AI chat (Phase 3 — deferred) | [`epics/phase-3-monetisation-ai.md`](epics/phase-3-monetisation-ai.md) → [`epics/epic-04-ai-assistant.md`](epics/epic-04-ai-assistant.md) → [`prd/06-features-ai-chat.md`](prd/06-features-ai-chat.md) | ~300 |
| Info / legal / launch (Epic 5) | [`epics/epic-05-launch.md`](epics/epic-05-launch.md) → [`prd/07-features-info.md`](prd/07-features-info.md) | ~150 |
| **Phase 1 improvements (Epic 6)** | [`epics/epic-06-phase1-improvements.md`](epics/epic-06-phase1-improvements.md) → [`prd/16-phase1-improvements.md`](prd/16-phase1-improvements.md) | ~200 |
| **Phase 1 UX wave 2 (Epic 7)** | [`epics/epic-07-phase1-ux-wave2.md`](epics/epic-07-phase1-ux-wave2.md) → [`prd/17-phase1-improvements-wave2.md`](prd/17-phase1-improvements-wave2.md) | ~250 |
| App foundation (Epic 1) | [`epics/epic-01-foundation.md`](epics/epic-01-foundation.md) → [`../planning-artifacts/architecture.md`](../planning-artifacts/architecture.md) | ~200 |
| Design tokens / components | [`ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md`](ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md) (Phase 1) · Phase 2: [`ux-designs/ux-japan_winter_sport-2026-06-23/DESIGN.md`](ux-designs/ux-japan_winter_sport-2026-06-23/DESIGN.md) | varies |
| User flows / personas | [`ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md`](ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md) (Phase 1) · Phase 2: [`ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md`](ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md) | ~350 |
| Analytics events | [`../../docs/tracking-plan.md`](../../docs/tracking-plan.md) (authoritative) → [`prd/15-analytics-legal-glossary.md`](prd/15-analytics-legal-glossary.md) |
| Quiz scoring | [`../../docs/quiz-scoring.md`](../../docs/quiz-scoring.md) | ~100 |
| Resort content / markdown | [`../../content/CONTENT_MODEL.md`](../../content/CONTENT_MODEL.md) → [`prd/10-content-and-decisions.md`](prd/10-content-and-decisions.md) | varies |
| Implementation (current sprint) | [`../implementation-artifacts/sprint-status.yaml`](../implementation-artifacts/sprint-status.yaml) + story file for your ticket | varies |

---

## Document map

### Sharded PRD (`prd/`)

| File | PRD source | Purpose |
|------|------------|---------|
| [`prd/README.md`](prd/README.md) | — | PRD shard index |
| [`prd/01-overview-and-goals.md`](prd/01-overview-and-goals.md) | §1–2 | Vision, users, success metrics |
| [`prd/02-phase1-functional-requirements.md`](prd/02-phase1-functional-requirements.md) | §3 FR-1–7 | One-page FR checklist |
| [`prd/03-features-discovery.md`](prd/03-features-discovery.md) | §3.1, FR-1 | Home, Explore, 20 resorts |
| [`prd/04-features-quiz-onboarding.md`](prd/04-features-quiz-onboarding.md) | FR-4, FR-5, Screen 0/0b | Quiz + onboarding |
| [`prd/05-features-detail-getting-there.md`](prd/05-features-detail-getting-there.md) | §3.2, §3.6, FR-2/3 | Detail page + transport |
| [`prd/06-features-ai-chat.md`](prd/06-features-ai-chat.md) | FR-6, §4.2a | Resort Q&A chat |
| [`prd/07-features-info.md`](prd/07-features-info.md) | FR-7 | Info tab, legal |
| [`prd/08-ai-content.md`](prd/08-ai-content.md) | §4 | AI strategy, guardrails, freshness |
| [`prd/09-security-and-architecture.md`](prd/09-security-and-architecture.md) | §6–7 | Security + stack summary |
| [`prd/10-content-and-decisions.md`](prd/10-content-and-decisions.md) | §8, §11 | Content schema pointers, resolved decisions |
| [`prd/11-roadmap-scope.md`](prd/11-roadmap-scope.md) | §9–10 | Out of scope, release gates |
| [`prd/12-ux-theme-and-layout.md`](prd/12-ux-theme-and-layout.md) | §12.0–12.4 | Theme A tokens, typography |
| [`prd/13-ux-screens.md`](prd/13-ux-screens.md) | §12.5 | Screen-by-screen specs |
| [`prd/14-ux-components-motion.md`](prd/14-ux-components-motion.md) | §12.6–12.11 | Components, motion, a11y |
| [`prd/15-analytics-legal-glossary.md`](prd/15-analytics-legal-glossary.md) | §13, §17–18 | Analytics principles, legal, glossary |
| [`prd/16-phase1-improvements.md`](prd/16-phase1-improvements.md) | §3.8 Epic 6 | Post-build UX/content improvements |
| [`prd/17-phase1-improvements-wave2.md`](prd/17-phase1-improvements-wave2.md) | §3.9 Epic 7 | Post-launch UX wave 2 |

**Full source:** [`prd.md`](prd.md) (1,465 lines) — use when shards reference § sections not duplicated here.

### Sharded epics (`epics/`)

| File | Purpose |
|------|---------|
| [`epics/README.md`](epics/README.md) | Epic shard index |
| [`epics/requirements-inventory.md`](epics/requirements-inventory.md) | FR/NFR/UX-DR inventory + coverage map |
| [`epics/epic-01-foundation.md`](epics/epic-01-foundation.md) | Stories 1.1–1.7 |
| [`epics/epic-02-discover.md`](epics/epic-02-discover.md) | Stories 2.1–2.7 (includes quiz) |
| [`epics/epic-03-detail.md`](epics/epic-03-detail.md) | Stories 3.1–3.5 (Phase 1); 3.6 → Phase 3 |
| [`epics/phase-3-monetisation-ai.md`](epics/phase-3-monetisation-ai.md) | Story 3.6 + Epic 4 — deferred |
| [`epics/epic-04-ai-assistant.md`](epics/epic-04-ai-assistant.md) | Stories 4.1–4.4 (Phase 3) |
| [`epics/epic-05-launch.md`](epics/epic-05-launch.md) | Stories 5.1–5.5 |
| [`epics/epic-06-phase1-improvements.md`](epics/epic-06-phase1-improvements.md) | Stories 6.1–6.12 (Phase 1.1) |
| [`epics/epic-07-phase1-ux-wave2.md`](epics/epic-07-phase1-ux-wave2.md) | Stories 7.1–7.6 (Phase 1.2) |
| [**`epics/epics-phase2.md`**](epics/epics-phase2.md) | **Phase 2 master** — Stories 9.1–15.4 (27 stories) |
| [`epics/epic-09-supabase-auth.md`](epics/epic-09-supabase-auth.md) | Stories 9.1–9.5 (FR-8) |
| [`epics/epic-10-nav-saved.md`](epics/epic-10-nav-saved.md) | Stories 10.1–10.4 (FR-15, FR-16) |
| [`epics/epic-11-resort-map.md`](epics/epic-11-resort-map.md) | Stories 11.1–11.3 (FR-9) |
| [`epics/epic-12-community-reviews.md`](epics/epic-12-community-reviews.md) | Stories 12.1–12.4 (FR-10) — **reviews shipped Phase 2** |
| [`epics/epic-13-passport-profile.md`](epics/epic-13-passport-profile.md) | Stories 13.1–13.4 (FR-11, FR-17) |
| [`epics/epic-14-trip-planning.md`](epics/epic-14-trip-planning.md) | Stories 14.1–14.3 (FR-12) |
| [`epics/epic-15-phase2-launch.md`](epics/epic-15-phase2-launch.md) | Stories 15.1–15.4 (FR-13, FR-14, launch) |
| [`epics/phase-3-user-community.md`](epics/phase-3-user-community.md) | Review **AI theme summaries** only (Phase 3); core UGC → Epic 12 |

**Full source:** [`epics.md`](epics.md) (657 lines)

### Feature briefs (`features/`)

Self-contained implementation packets for a single capability. **Start here** when beginning a new agent session on that feature.

| File | Covers |
|------|--------|
| [`features/README.md`](features/README.md) | How to write/use feature briefs |
| [`features/quiz.md`](features/quiz.md) | Story 2.7 / FR-4 — Find My Resort quiz |

### Other authoritative docs (not sharded)

| Doc | When to read |
|-----|--------------|
| [`architecture.md`](architecture.md) | Stack decisions, folder layout, cross-cutting patterns |
| [`ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md`](ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md) | Theme A tokens (Phase 1) |
| [`ux-designs/ux-japan_winter_sport-2026-06-23/DESIGN.md`](ux-designs/ux-japan_winter_sport-2026-06-23/DESIGN.md) | Phase 2 component tokens + mocks |
| [`ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md`](ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md) | IA, flows (Phase 1) |
| [`ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md`](ux-designs/ux-japan_winter_sport-2026-06-23/EXPERIENCE.md) | Phase 2 IA, flows (UJ-1–5), states |
| [`../../docs/tracking-plan.md`](../../docs/tracking-plan.md) | Event names/properties — **authoritative for analytics** |
| [`../../content/CONTENT_MODEL.md`](../../content/CONTENT_MODEL.md) | Resort markdown schema |
| [`../../web/AGENTS.md`](../../web/AGENTS.md) | Next.js app conventions |

---

## Adding new feature briefs

When starting a new epic story, copy the structure in [`features/quiz.md`](features/quiz.md):

- Goal + user story in one paragraph
- Acceptance criteria (from epics)
- UX spec essentials (from PRD §12.5 + DESIGN.md)
- Architecture/file paths
- Analytics events (from tracking-plan)
- Dependencies + deferred items
- Links to shards (not full monoliths)

---

*Generated from PRD v1.1 and epics breakdown. Phase 1 scope updated 2026-06-13 — FR-6/AI deferred to Phase 3.*
