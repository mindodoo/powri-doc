<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 13: Ski Passport & Public Profile

**Stories:** 13.1–13.4 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-11, FR-17 · **NFRs:** NFR-11, 12  
**Depends on:** Epic 9, Epic 12.2 (optional, for reviewer badges)  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 13

---

## Overview

| Story | Summary |
|-------|---------|
| **13.1** | Manual check-in + optional passport photo |
| **13.2** | Badge rules engine + unlock toast (16 badges) |
| **13.3** | Owner passport view `/passport` |
| **13.4** | Public profile `/u/[username]` with noindex |

---

## Recommended order

```
13.1 → 13.2 → 13.3 → 13.4
```

13.2 can partially ship with 13.1 (check-in badges); review badges need 12.2.

---

## Key paths

| Area | Path |
|------|------|
| Badges | `web/src/lib/badges/definitions.ts`, `evaluate.ts`, `regions.ts` |
| Passport API | `web/src/app/api/passport/` |
| Passport UI | `web/src/components/passport/` |
| Profile | `web/src/app/[locale]/u/[username]/page.tsx` |

---

## Validate

- [ ] Check-in → First Tracks badge toast
- [ ] Hokkaido Explorer after 3 Hokkaido resorts (region map)
- [ ] Public profile shows badges; no PII leak
