<!-- generated from epics/phase2/epics-phase2.md 2026-07-16 — do not edit; run prd-shard-regen.md -->

# Epic 11: Resort Map & Surroundings

**Stories:** 11.1–11.3 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-9 · **NFRs:** NFR-15, 16  
**Depends on:** Epic 9.1 (places_cache table) · **11.2–11.3 blocked on 11.1**  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 11

---

## Overview

| Story | Summary |
|-------|---------|
| **11.1** | `latitude`/`longitude` + optional `curated_map_places` in CONTENT_MODEL + schema + all 20 resorts |
| **11.2** | `/api/places/nearby` merging curated + Google Places (dedup by `place_id`) + `places_cache` (configurable TTL) + rate limit |
| **11.3** | Leaflet map section on resort detail — curated picks first with "Powri pick" marker + lazy load + attribution |

**No login required** — public feature.

**Curated places (`curated_map_places`):** PO-maintained editorial list of hand-picked places (ski-in/ski-out & other accommodation, restaurants, transport, etc.) stored in resort frontmatter, rendered before Google Places API results. Optional per resort — empty array is a fully-supported, non-blocking state (API-only fallback). See `input_docs/story-curated-map-places.md` for the detailed schema, category enum, and PO editorial workflow.

**Cost control (corrected 2026-07-16):** Google discontinued the pooled $200/mo credit in March 2025 — replaced by per-SKU monthly free-call caps + a one-time, non-recurring $300 trial credit for new billing accounts. This epic's Nearby Search field mask is **Pro-tier only** (no `rating`/`userRatingCount`, which would trigger the pricier Enterprise SKU) and the `places_cache` TTL is **configurable** (env var `PLACES_CACHE_TTL_HOURS`, default 168h/7d, minimum 24h). At 20 resorts × 7 API-fillable categories, worst case stays under the Pro free cap (5,000 calls/mo) at either TTL setting — expected steady-state cost **$0/month**. See addendum §F for full math.

---

## Recommended order

```
11.1 (content + schema, incl. curated places) → 11.2 → 11.3
```

---

## Key paths

| Area | Path |
|------|------|
| Content schema | `content/content-model.md`, `web/src/lib/content/schema.ts`, `web/src/lib/content/types.ts` |
| Places API | `web/src/app/api/places/nearby/route.ts` |
| Map UI | `web/src/components/map/ResortMapSection.tsx` |
| Detail | `web/src/components/resort/ResortDetailContent.tsx` |
| CSP | `web/src/lib/security/csp.ts` |

---

## Validate

- [ ] All 20 resorts have coords; build passes
- [ ] `curated_map_places` schema validates (warn on bad `place_id`/`highlight`/`tags`, fail on bad `category` enum); empty array does not warn
- [ ] Map lazy-loads; no CWV regression on detail LCP
- [ ] Curated places render first, deduplicated against API results by `place_id`; "Powri pick" marker visible
- [ ] Google attribution visible; API key not in client bundle
- [ ] Nearby Search field mask is Pro-tier only — no `rating`/`userRatingCount` in the request or response
- [ ] `places_cache` TTL reads from `PLACES_CACHE_TTL_HOURS` (default 168h/7d, minimum 24h) — not hardcoded
- [ ] GCP billing budget alert configured on the Places API project
