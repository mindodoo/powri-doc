# Epic 12: Community Reviews & Comments

**Stories:** 12.1–12.4 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-10 · **NFRs:** NFR-10, 11, 12  
**Depends on:** Epic 9 complete · **Legal:** Story 15.3 before public launch  
**Full ACs:** [`epics-phase2.md`](epics-phase2.md) — Epic 12

> **Note:** Moved from Phase 3 [`phase-3-user-community.md`](phase-3-user-community.md) per Phase 2 PRD.

---

## Overview

| Story | Summary |
|-------|---------|
| **12.1** | Experience section + SSR aggregate + client review list |
| **12.2** | Review submit/edit + up to 5 photos |
| **12.3** | Flat comment thread |
| **12.4** | Report + admin hide API |

---

## Recommended order

```
12.1 → 12.2 → 12.3 → 12.4
```

---

## Key paths

| Area | Path |
|------|------|
| Experience UI | `web/src/components/experience/` |
| Reviews API | `web/src/app/api/reviews/` |
| Comments API | `web/src/app/api/comments/` |
| Admin | `web/src/app/api/admin/` |
| Sanitize | `web/src/lib/ugc/sanitize.ts` |

---

## Validate

- [ ] Guest reads reviews without login
- [ ] One review per user per resort; edit works
- [ ] Report → admin hide → content disappears
- [ ] Photo limits enforced client + server
