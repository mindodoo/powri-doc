<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.2: Resort Map & Surroundings (FR-9)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.2 · **See also:** **Epic:** [`epic-11-resort-map.md`](../../../epics/phase2/shards/epic-11-resort-map.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.2 Resort Map & Surroundings (FR-9)

**Description:** Each resort detail page gains an embedded **Map** section showing the resort location plus nearby points of interest via **Google Places API** (restaurants, onsen, convenience, attractions). Map **renders** with **Leaflet + OpenStreetMap tiles** (Phase 1 stack); POI data from Google Places server proxy. Phase 1 trail map link-out remains. Map is public — no login.

Realizes UJ-1.

#### FR-9.1: Resort map section

**Guest or User** can view an interactive map centered on the resort with category-filtered nearby places.

**Consequences:**
- Map section title: **"Around the resort"** — placed **below Getting There**, above the Experience section (see addendum §M).
- Resort center from seed content `latitude` / `longitude` (addendum §K) — not geocoded at runtime.
- Categories: at minimum `restaurant`, `lodging`, `spa` (onsen), `tourist_attraction`, `convenience`.
- Pin tap shows name, rating (if available), short address; "Open in Google Maps" external link.
- Google Places API called server-side via proxy route — API key never exposed client-side.
- Google Places attribution displayed per ToS (addendum §L).
- Graceful fallback if Places API fails: static resort pin + link to Google Maps search.
- Map lazy-loaded (intersection observer) — NFR-16.

#### FR-9.2: Cost control

**System** caches Places results per resort per category with TTL (24h minimum).

**Consequences:**
- Estimated cost at 20 resorts × low traffic: well within Google $200/mo free credit `[ASSUMPTION]`.
- Analytics: `map_section_viewed`, `map_pin_tapped` with `place_category`.

**Out of Scope Phase 2:**
- Accommodation booking or Powri-curated lodging recommendations.
- Custom editorial POI overrides (Phase 3+ if needed).

---
