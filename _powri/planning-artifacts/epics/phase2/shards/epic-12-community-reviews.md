<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 12: Community Reviews & Comments

**Stories:** 12.1–12.4 · **Status:** Done · **Phase:** 2  
**FRs:** FR-10 · **NFRs:** NFR-10, 11, 12  
**Depends on:** Epic 9 complete · **Legal:** Story 15.3 before public launch  
**Retrospective:** [`epic-12-retro-2026-08-13.md`](../../../../implementation-artifacts/Epic-Retro/epic-12-retro-2026-08-13.md)  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 12

> **Note:** Moved from Phase 3 [`phase-3-user-community.md`](../../phase1/shards/phase-3-user-community.md) per Phase 2 PRD.

---

## PO decision — Story 12.3 deferred (2026-08-13)

**Decision:** Story **12.3** (flat comment thread under reviews) is **deferred**. We no longer want a comment section on resort detail under the Experience section.

**Rationale:** Comment-on-resort UX is replaced by a different direction — connecting snow riders with each other through another mechanism (feature TBD; not scoped in Epic 12).

**Impact on Epic 12:**

| Story | Status | Notes |
|-------|--------|-------|
| 12.1 | done | Experience section + review list |
| 12.2 | done | Submit/edit review + photos |
| 12.2.1 | done | R2 photo storage |
| **12.3** | **deferred** | Flat comment thread — out of Phase 2 |
| **12.4** | done | Report review + admin hide API |

**Implementation order (updated):**

```
12.1 → 12.2 → 12.2.1 → 12.4
(12.3 deferred — revisit when snow-rider connection feature is defined)
```

**For next agents:** Do not implement `CommentList`, `/api/comments`, comment report/hide API, or "Add a comment" CTAs. Story **12.4** ships **reviews-only** moderation (PO confirmed 2026-08-13). Revisit when snow-rider community feature is researched and scoped.

---

## Overview

| Story | Summary |
|-------|---------|
| **12.1** | Experience section + SSR aggregate + client review list |
| **12.2** | Review submit/edit + up to 5 photos |
| **12.2.1** | Review photo storage — R2 migration + WebP compression |
| **12.3** | ~~Flat comment thread~~ **deferred** (2026-08-13) |
| **12.4** | Report + admin hide API |

---

## Key paths

| Area | Path |
|------|------|
| Experience UI | `web/src/components/experience/` |
| Reviews API | `web/src/app/api/reviews/` |
| Comments API | `web/src/app/api/comments/` *(deferred with 12.3)* |
| Admin | `web/src/app/api/admin/` |
| Sanitize | `web/src/lib/ugc/sanitize.ts` |

---

## Validate

- [x] Guest reads reviews without login
- [x] One review per user per resort; edit works
- [x] Report → admin hide → content disappears
- [x] Photo limits enforced client + server
- [ ] ~~Flat comment thread~~ deferred
