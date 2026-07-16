<!-- generated from prds/phase2/prd-phase2.md 2026-07-16 — do not edit -->

# PRD §5.2: Resort Map & Surroundings (FR-9)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.2 · **See also:** **Epic:** [`epic-11-resort-map.md`](../../../epics/phase2/shards/epic-11-resort-map.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.2 Resort Map & Surroundings (FR-9)

**Description:** Each resort detail page gains an embedded **Map** section showing the resort location plus nearby points of interest. POI data blends a **PO-curated editorial list** (`curated_map_places` in resort content, rendered first) with **Google Places API** results (server proxy, fills remaining category slots). Map **renders** with **Leaflet + OpenStreetMap tiles** (Phase 1 stack). Phase 1 trail map link-out remains. Map is public — no login.

Realizes UJ-1.

#### FR-9.1: Resort map section

**Guest or User** can view an interactive map centered on the resort with category-filtered nearby places, curated picks shown first.

**Consequences:**
- Map section title: **"Around the resort"** — placed **below Getting There**, above the Experience section (see addendum §M).
- Resort center from seed content `latitude` / `longitude` (addendum §K) — not geocoded at runtime.
- Categories: `restaurant`, `accommodation` (incl. `ski_in_ski_out`), `supermarket`, `convenience`, `transport`, `onsen`/spa, `attraction`. `ski_in_ski_out` is editorial-only — no Google Places equivalent, never API-filled.
- Rendering order: curated places (resort frontmatter `curated_map_places`) first, then Google Places API results per category; deduplicated by `place_id` (curated entry wins on collision).
- Curated pins carry a distinct "Powri pick" visual marker — exact treatment TBD by UX.
- Pin tap shows name, short address/editorial highlight; "Open in Google Maps" external link. **No Google `rating` field** — intentionally excluded, see FR-9.2 cost control.
- Google Places API called server-side via proxy route — API key never exposed client-side.
- Google Places attribution displayed per ToS (addendum §L).
- Graceful fallback if Places API fails: curated places (if any) + static resort pin + link to Google Maps search.
- Map lazy-loaded (intersection observer) — NFR-16.

#### FR-9.2: Cost control

**System** caches Places results per resort per category with a **configurable TTL** (env var, default **7 days / 168h**, minimum **24h**) — operators can drop to 24h for periods of higher POI churn (e.g. end-of-season closures) without a code change. Curated places reduce API-fill volume per category — no DB/API call needed for editorial entries. The Nearby Search request's field mask is restricted to **Pro-tier fields only** (name, formatted address, location, photos, type) — Google's `rating` / `userRatingCount` fields are never requested, because Google bills the entire call at the highest SKU any requested field belongs to, and rating triggers the pricier Enterprise SKU.

**Consequences:**
- **`[RESOLVED]`** Google discontinued the pooled $200/mo credit in March 2025 (replaced by per-SKU monthly free-call caps + a one-time, non-recurring $300 trial credit for new billing accounts — not a monthly pool). With the Pro-tier-only field mask above, this feature's Nearby Search calls qualify for the **Pro** free cap: **5,000 calls/month**.
- Cost math: 20 resorts × 7 API-fillable categories (`ski_in_ski_out` is editorial-only, never API-filled) = 140 resort+category cache slots. At the default 7-day TTL, worst case ≈ 600 calls/month; even at the minimum 24h TTL, worst case ≈ 4,200 calls/month — both comfortably under the 5,000/mo Pro free cap regardless of the TTL setting chosen. Expected steady-state cost: **$0/month**.
- A GCP billing budget alert is configured on the Places API project as a guardrail, since this is a real (if currently $0-expected) recurring cost line rather than a "free" assumption.
- Analytics: `map_section_viewed`, `map_pin_tapped` with `place_category` `[OPEN ITEM: consider adding place_source: curated|api]`.

**Out of Scope Phase 2:**
- Accommodation **booking** (reservations/pricing) — curated accommodation/ski-in-ski-out *recommendations* are in scope.
- Bulk/automated curation tooling — editorial entries are added manually by the PO, one resort at a time.
- Populating `curated_map_places` for all 20 resorts is **not a ship-blocking gate** — an empty array is a fully-supported state (API-only fallback).

---
