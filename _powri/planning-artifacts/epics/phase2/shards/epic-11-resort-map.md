<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 11: Resort Map & Surroundings

**Stories:** 11.1–11.3 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-9 · **NFRs:** NFR-15, 16  
**Depends on:** Epic 9.1 (places_cache table) · **11.2–11.3 blocked on 11.1**  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 11

---

## Overview

| Story | Summary |
|-------|---------|
| **11.1** | `latitude`/`longitude` in CONTENT_MODEL + schema + all 20 resorts |
| **11.2** | `/api/places/nearby` + `places_cache` + rate limit |
| **11.3** | Leaflet map section on resort detail + lazy load + attribution |

**No login required** — public feature.

---

## Recommended order

```
11.1 (content + schema) → 11.2 → 11.3
```

---

## Key paths

| Area | Path |
|------|------|
| Content schema | `content/content-model.md`, `web/src/lib/content/schema.ts` |
| Places API | `web/src/app/api/places/nearby/route.ts` |
| Map UI | `web/src/components/map/ResortMapSection.tsx` |
| Detail | `web/src/components/resort/ResortDetailContent.tsx` |
| CSP | `web/src/lib/security/csp.ts` |

---

## Validate

- [ ] All 20 resorts have coords; build passes
- [ ] Map lazy-loads; no CWV regression on detail LCP
- [ ] Google attribution visible; API key not in client bundle
