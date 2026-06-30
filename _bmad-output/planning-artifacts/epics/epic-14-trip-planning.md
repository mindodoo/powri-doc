# Epic 14: Trip Planning Skeleton

**Stories:** 14.1–14.3 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-12 · **NFRs:** NFR-11  
**Depends on:** Epic 9 complete  
**Full ACs:** [`epics-phase2.md`](epics-phase2.md) — Epic 14

> AI trip fill is **Phase 3** — this epic ships template skeleton + disabled "Fill with AI" stub.

---

## Overview

| Story | Summary |
|-------|---------|
| **14.1** | Trip CRUD + template generation + 5-trip quota |
| **14.2** | Plan tab list + logged-out empty state |
| **14.3** | Trip detail timeline + edit activities |

---

## Recommended order

```
14.1 → 14.2 → 14.3
```

---

## Key paths

| Area | Path |
|------|------|
| Templates | `web/src/lib/trips/templates.ts`, `types.ts` |
| Trips API | `web/src/app/api/trips/` |
| Plan UI | `web/src/components/plan/` |
| Routes | `web/src/app/[locale]/plan/` |

---

## Validate

- [ ] 6th trip creation blocked with clear message
- [ ] Template matches date range (arrival → ski days → departure)
- [ ] Trip data not indexed; requires login to edit
