<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 10: Navigation & Saved Resorts

**Stories:** 10.1–10.4 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-15, FR-16 · **NFR:** NFR-17  
**Depends on:** Epic 9.2 (session for sync)  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 10

---

## Overview

| Story | Summary |
|-------|---------|
| **10.1** | 4-tab bottom nav + DesktopTopNav at 768px |
| **10.2** | Hamburger overflow: Passport, Info, Account (no bottom-nav dupes) |
| **10.3** | Heart button — guest localStorage + user API |
| **10.4** | `/saved` page + merge on login + banner for guests |

---

## Recommended order

```
10.1 → 10.2 → 10.3 → 10.4
```

10.1 and 10.2 can ship together (same layout files).

---

## Key paths

| Area | Path |
|------|------|
| Bottom nav | `web/src/components/layout/BottomNav.tsx` |
| Desktop nav | `web/src/components/layout/DesktopTopNav.tsx` (new) |
| Menu | `web/src/components/layout/AppMenuSheet.tsx` |
| Shell | `web/src/components/layout/AppShell.tsx` |
| Saved | `web/src/components/saved/`, `web/src/app/[locale]/saved/` |
| API | `web/src/app/api/saved/` |

---

## Validate

- [ ] 375px: 4 bottom tabs visible; hamburger has Passport not Home
- [ ] 1024px: top nav only; no bottom nav
- [ ] Guest save → login → merge → saves on second device after sync
