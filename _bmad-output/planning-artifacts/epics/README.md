# Epic Shards

Story-level extracts from [`../epics.md`](../epics.md) (Phase 1) and [`epics-phase2.md`](epics-phase2.md) (Phase 2).

**When to use:** Read the epic shard for your sprint. Use [`requirements-inventory.md`](requirements-inventory.md) for FR/NFR cross-reference. Use [`../features/`](../features/) for single-feature agent sessions.

## Phase 1 (complete)

| Shard | Stories | FRs |
|-------|---------|-----|
| [requirements-inventory.md](requirements-inventory.md) | — | FR-1–7, FR-8–17 (Phase 2) |
| [epic-01-foundation.md](epic-01-foundation.md) | 1.1–1.7 | Enables all |
| [epic-02-disciscover.md](epic-02-discover.md) | 2.1–2.7 | FR-1, FR-4, FR-5 |
| [epic-03-detail.md](epic-03-detail.md) | 3.1–3.5 | FR-2, FR-3 |
| [phase-3-monetisation-ai.md](phase-3-monetisation-ai.md) | 3.6 + Epic 4 | FR-6 (deferred) |
| [epic-04-ai-assistant.md](epic-04-ai-assistant.md) | 4.1–4.4 | FR-6 (Phase 3) |
| [epic-05-launch.md](epic-05-launch.md) | 5.1–5.5 | FR-7, NFR-13 |
| [epic-06-phase1-improvements.md](epic-06-phase1-improvements.md) | 6.1–6.12 | IMP-1…IMP-12 |
| [epic-07-phase1-ux-wave2.md](epic-07-phase1-ux-wave2.md) | 7.1–7.6 | IMP-13…IMP-18 |
| [phase-3-user-community.md](phase-3-user-community.md) | — | Review AI summaries (Phase 3); **reviews → Epic 12** |

## Phase 2 (backlog)

| Shard | Stories | FRs |
|-------|---------|-----|
| [**epics-phase2.md**](epics-phase2.md) | **9.1–15.4** (27 stories) | **Full ACs** |
| [epic-09-supabase-auth.md](epic-09-supabase-auth.md) | 9.1–9.5 | FR-8 |
| [epic-10-nav-saved.md](epic-10-nav-saved.md) | 10.1–10.4 | FR-15, FR-16 |
| [epic-11-resort-map.md](epic-11-resort-map.md) | 11.1–11.3 | FR-9 |
| [epic-12-community-reviews.md](epic-12-community-reviews.md) | 12.1–12.4 | FR-10 |
| [epic-13-passport-profile.md](epic-13-passport-profile.md) | 13.1–13.4 | FR-11, FR-17 |
| [epic-14-trip-planning.md](epic-14-trip-planning.md) | 14.1–14.3 | FR-12 |
| [epic-15-phase2-launch.md](epic-15-phase2-launch.md) | 15.1–15.4 | FR-13, FR-14 |

**Implementation order:** 9 → 10 → 11 (needs lat/lng) · 12 (after 9) → 13 → 14 → 15

**Sprint status:** [`../../implementation-artifacts/sprint-status.yaml`](../../implementation-artifacts/sprint-status.yaml)  
**Story artifacts:** [`../../implementation-artifacts/`](../../implementation-artifacts/) (one `.md` per completed story)
