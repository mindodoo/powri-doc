<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 15: Phase 2 Launch, Analytics & Compliance

**Stories:** 15.1–15.4 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-13, FR-14, FR-7 (update) · **NFRs:** NFR-8  
**Depends on:** Epics 9–14 feature-complete  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 15

---

## Overview

| Story | Summary |
|-------|---------|
| **15.1** | AI entry stubs on quiz results + resort detail |
| **15.2** | `phase2Events.ts` + tracking-plan §3 sync |
| **15.3** | Privacy, Terms, community guidelines |
| **15.4** | Release gates + SEO verification |

---

## Recommended order

```
15.2 (can start early alongside features) → 15.1 → 15.3 → 15.4
```

15.3 must complete before UGC goes to production users.

---

## Key paths

| Area | Path |
|------|------|
| AI stubs | resort detail + `QuizResultsReveal` |
| Analytics | `web/src/lib/analytics/phase2Events.ts`, `docs/analytics/tracking-plan.md` |
| Legal | `web/src/app/[locale]/privacy/`, `terms/` |

---

## Validate

- [ ] All PRD §13 release gates checked
- [ ] PostHog live events for Phase 2 funnel
- [ ] Organic resort page crawl test unchanged
