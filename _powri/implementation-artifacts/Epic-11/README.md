# Epic 11: Resort Map & Surroundings

Interactive **"Around the resort"** map on each resort detail page — PO-curated picks first, Google Places API fill, Mapbox + category pins. Public; no login.

**FRs:** FR-9 · **Depends on:** Epic 9.1 (`places_cache` table)  
**Status:** Done (2026-07-17) · **Retrospective:** [`Epic-Retro/epic-11-retro-2026-07-17.md`](../Epic-Retro/epic-11-retro-2026-07-17.md)

## Stories

| # | Summary |
|---|---------|
| **11.1** | Geo fields + `curated_map_places` content pipeline (schema, validation, all 20 resorts) |
| **11.2** | `/api/places/nearby` — curated + Google Places merge, `places_cache`, rate limit |
| **11.3** | `ResortMapSection` UI — lazy load, category filters, Powri pick marker, attribution |
| **11.4** | Bug fix: Google Maps URL + required `place_name` on curated entries |
| **11.5** | Mapbox migration (Leaflet → Mapbox Light + `react-map-gl`) |
| **11.6** | Per-category pin shapes (lucide icons + `categoryVisuals.ts`) |
| **11.7** | All/single-select chips + `ski_resort` curated category |
| **11.8** | Nearby radius 3 km / max 8 + auto-fit-bounds |
| **11.9** | Desktop split-view rail + chip-gated POI lists |

**Original order:** 11.1 → 11.2 → 11.3 · **Follow-ups:** 11.4–11.9 (PO polish after manual QA)
