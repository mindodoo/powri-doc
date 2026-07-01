# Epic shards

Story-level extracts from monoliths. **Epic numbers continue across phases** (1–8 Phase 1, 9–15 Phase 2).

| Monolith (edit here) | Shards (read-only) |
|----------------------|-------------------|
| [`phase1/epics-phase1.md`](phase1/epics-phase1.md) | [`phase1/shards/`](phase1/shards/) |
| [`phase2/epics-phase2.md`](phase2/epics-phase2.md) | [`phase2/shards/`](phase2/shards/) |

After editing a monolith, follow [`../prd-shard-regen.md`](../prd-shard-regen.md).

**FR cross-reference:** [`requirements-inventory.md`](requirements-inventory.md)  
**Feature briefs:** [`../features/`](../features/)

## Phase 1 (complete)

| Shard | Stories | FRs |
|-------|---------|-----|
| [requirements-inventory.md](requirements-inventory.md) | — | FR-1–7, FR-8–17 (Phase 2) |
| [epic-01-foundation.md](phase1/shards/epic-01-foundation.md) | 1.1–1.7 | Enables all |
| [epic-02-discover.md](phase1/shards/epic-02-discover.md) | 2.1–2.7 | FR-1, FR-4, FR-5 |
| [epic-03-detail.md](phase1/shards/epic-03-detail.md) | 3.1–3.5 | FR-2, FR-3 |
| [phase-3-monetisation-ai.md](phase1/shards/phase-3-monetisation-ai.md) | 3.6 + Epic 4 | FR-6 (deferred) |
| [epic-04-ai-assistant.md](phase1/shards/epic-04-ai-assistant.md) | 4.1–4.4 | FR-6 (Phase 3) |
| [epic-05-launch.md](phase1/shards/epic-05-launch.md) | 5.1–5.5 | FR-7, NFR-13 |
| [epic-06-phase1-improvements.md](phase1/shards/epic-06-phase1-improvements.md) | 6.1–6.12 | IMP-1…IMP-12 |
| [epic-07-phase1-ux-wave2.md](phase1/shards/epic-07-phase1-ux-wave2.md) | 7.1–7.6 | IMP-13…IMP-18 |
| [epic-08-quiz-cosmic-story.md](phase1/shards/epic-08-quiz-cosmic-story.md) | 8.1–8.4 | FR-4a–j (Cosmic v2) |
| [phase-3-user-community.md](phase1/shards/phase-3-user-community.md) | — | Review AI summaries (Phase 3) |

## Phase 2 (backlog)

| Shard | Stories | FRs |
|-------|---------|-----|
| [**epics-phase2.md**](phase2/epics-phase2.md) | **9.1–15.4** (27 stories) | **Full ACs** |
| [epic-09-supabase-auth.md](phase2/shards/epic-09-supabase-auth.md) | 9.1–9.5 | FR-8 |
| [epic-10-nav-saved.md](phase2/shards/epic-10-nav-saved.md) | 10.1–10.4 | FR-15, FR-16 |
| [epic-11-resort-map.md](phase2/shards/epic-11-resort-map.md) | 11.1–11.3 | FR-9 |
| [epic-12-community-reviews.md](phase2/shards/epic-12-community-reviews.md) | 12.1–12.4 | FR-10 |
| [epic-13-passport-profile.md](phase2/shards/epic-13-passport-profile.md) | 13.1–13.4 | FR-11, FR-17 |
| [epic-14-trip-planning.md](phase2/shards/epic-14-trip-planning.md) | 14.1–14.3 | FR-12 |
| [epic-15-phase2-launch.md](phase2/shards/epic-15-phase2-launch.md) | 15.1–15.4 | FR-13, FR-14 |

**Implementation order:** 9 → 10 → 11 (needs lat/lng) · 12 (after 9) → 13 → 14 → 15

**Sprint status:** [`_powri/implementation-artifacts/sprint-status.yaml`](../../implementation-artifacts/sprint-status.yaml)  
**Story artifacts:** [`_powri/implementation-artifacts/`](../../implementation-artifacts/)
