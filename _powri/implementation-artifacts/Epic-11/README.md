# Epic 11: Resort Map & Surroundings

Interactive **"Around the resort"** map on each resort detail page — PO-curated picks first, Google Places API fill, Leaflet + OSM tiles. Public; no login.

**FRs:** FR-9 · **Depends on:** Epic 9.1 (`places_cache` table)

## Stories

| # | Summary |
|---|---------|
| **11.1** | Geo fields + `curated_map_places` content pipeline (schema, validation, all 20 resorts) |
| **11.2** | `/api/places/nearby` — curated + Google Places merge, `places_cache`, rate limit |
| **11.3** | `ResortMapSection` UI — lazy load, category filters, Powri pick marker, attribution |

**Order:** 11.1 → 11.2 → 11.3
