baseline_commit: f7bb545

# Story 11.3: Resort Map Section UI

Status: done

<!-- Ultimate context engine analysis completed - comprehensive developer guide created -->
<!-- PO decisions 2026-07-17: see § PO decisions -->

## Story

As a **visitor on resort detail**,  
I want an "Around the resort" map with category pins, curated picks shown first and visually distinguished,  
so that I discover nearby food, onsen, attractions, and PO-recommended spots.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) · UX [`ux-designs/ux-phase2/design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) + [`experience.md`](../../planning-artifacts/ux-designs/ux-phase2/experience.md) + mock [`mockups/key-resort-detail-phase2.html`](../../planning-artifacts/ux-designs/ux-phase2/mockups/key-resort-detail-phase2.html) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.2** (`GET /api/places/nearby` — `{ curated, api, attribution }`) · Story **11.1** (resort `latitude`/`longitude`/`mapDefaultZoom`/`curatedMapPlaces`)

**Blocks:** Epic 12 Experience section placement (map must remain above Experience when 12.1 lands)

---

## PO decisions (2026-07-17)

| # | Topic | Decision |
|---|-------|----------|
| 1 | **Powri pick pin** | Black sparkle/star shape (not the blue resort pin / grey POI circle). Distinct from resort center and API pins. |
| 2 | **Category chips** | **7 chips** (PO said “6” but listed 7 labels — **labels win**). Accommodation chip **includes** `ski_in_ski_out` curated entries (no separate ski-in/ski-out chip). |
| 3 | **POI sheet fields** | **Name + vicinity/highlight only** — **no rating**. UX `design.md` “rating” line is stale vs PRD/addendum §L. |
| 4 | **Default filter** | **All chips selected** on first load (multi-select). Overrides UX `experience.md` “single-select” primitive for this story. |
| 5 | **`place_source`** | Include on `map_pin_tapped`: `curated` \| `api`. |
| 6 | **`place_category`** | Match **chip keys** (below) — not the old tracking-plan `restaurant\|onsen\|convenience\|other` collapse. |
| 7 | **POI list** | Visible list below map is **in scope** — curated **and** Google API-filled places (not a11y-only optional). |
| 8 | **API / Places failure fallback** | Follow PRD: curated (if any) + resort pin + **Google Maps search** link. Do **not** use UX “Map unavailable — try again later” as the primary Places-data failure UX (Leaflet/OSM still renders). |
| 9 | **Curated without lat/lng** | Still appear in the **list / sheet**; **omit map pin**. |

### Chip labels ↔ API `category` param(s)

| Chip label (UI) | Chip key (`place_category`) | API fetch(es) |
|-----------------|----------------------------|---------------|
| Restaurant | `restaurant` | `category=restaurant` |
| Accommodation | `accommodation` | `category=accommodation` **and** `category=ski_in_ski_out` (merge client-side; ski-in is curated-only) |
| Supermarket | `supermarket` | `category=supermarket` |
| Convenience store | `convenience` | `category=convenience` |
| Onsen | `onsen` | `category=onsen` |
| Attraction | `attraction` | `category=attraction` |
| Transportation | `transport` | `category=transport` |

---

## Acceptance Criteria

### A. Section placement & shell

1. **Given** resort detail  
   **When** the page renders  
   **Then** an **"Around the resort"** section appears **after Getting There** and **before** Practical tips (and before Experience when 12.1 exists)  
   **And** if `PracticalTipsBlock` currently lives inside `GettingThereSection`, extract it so this order is possible without putting the map below tips

2. **Given** the map section mounts  
   **When** it is not yet in viewport / Leaflet not loaded  
   **Then** show a skeleton placeholder at **`{components.map-section.height}` = 240px**, `{rounded.map}` = 12px (UX spine wins over mock’s 200px)

### B. Lazy load + analytics view

3. **Given** the visitor scrolls so the map section enters the viewport  
   **When** IntersectionObserver fires (once per page load; mirror `GettingThereSection` pattern, threshold ~0.35)  
   **Then** fire `map_section_viewed` with `resort_slug`  
   **And** dynamically import the Leaflet map (`next/dynamic`, `ssr: false`) — do not load Leaflet in the initial detail bundle before viewport entry (NFR-16)

### C. Map render (Leaflet + OSM)

4. **Given** Leaflet has loaded  
   **When** the map renders  
   **Then** center on resort `latitude`/`longitude` at `mapDefaultZoom` (default 13)  
   **And** OSM tiles are used (Phase 1 stack / architecture)  
   **And** a **resort center pin** uses `{colors.map-pin-resort}` / accent blue  
   **And** CSP allows OSM tile loading (`*.tile.openstreetmap.org` — update `web/src/lib/security/csp.ts` as needed; `img-src` already allows `https:` but prefer explicit tile host per architecture §5.1)

### D. Category chips (multi-select)

5. **Given** first paint of the map section  
   **When** chips render  
   **Then** all **7** chips are **selected**  
   **And** labels are exactly: Restaurant · Accommodation · Supermarket · Convenience store · Onsen · Attraction · Transportation

6. **Given** the visitor toggles chips  
   **When** the selection set changes  
   **Then** pins and the POI list refresh to the union of selected categories  
   **And** Accommodation always merges `accommodation` + `ski_in_ski_out` results when that chip is on

7. **Given** selected chips that need Google fill  
   **When** data is loaded  
   **Then** client calls `GET /api/places/nearby?resortSlug=&category=` per required API category (parallel OK; respect existing rate limit)  
   **And** never sends lat/lng as query params (server resolves from resort index)

### E. Pins, order, Powri pick

8. **Given** places for the active chip set  
   **When** pins render  
   **Then** curated places with lat/lng render first (per category / overall curated-before-api ordering)  
   **And** API places fill remaining pins  
   **And** client dedupes by `placeId` if merging multiple category responses (curated wins)  
   **And** API pins use `{colors.map-pin-poi}`  
   **And** curated pins use **black sparkle/star** DivIcon (PO decision) — not the default Leaflet marker

9. **Given** a curated entry **without** lat/lng  
   **When** places render  
   **Then** it appears in the **list** (and can open the sheet) but **has no map pin**

### F. POI list + sheet

10. **Given** loaded places  
    **When** the section is ready  
    **Then** a **visible list** below the map shows curated first, then API places, filtered to selected chips  
    **And** list rows are tappable (same sheet as pin tap)

11. **Given** the visitor taps a pin or list row  
    **When** the POI sheet opens  
    **Then** show:
    - **API place:** `name` + `vicinity`  
    - **Curated place:** primary text = `highlight` if non-empty, else **"Powri pick"** (schema has **no `name` / no vicinity** — do **not** call Place Details / Photos to invent a name) + optional `tags`  
    - curated badge/label **"Powri pick"** when `place_source=curated`  
    - **"Open in Google Maps"** external link via `placeId` (`https://www.google.com/maps/search/?api=1&query_place_id=…`)  
    **And** **never** show Google `rating` / review counts  
    **And** **no** Place Photos thumbnails (11.2 PO: `photoName` ignored in Phase 2 UI)

12. **Given** "Open in Google Maps" is tapped  
    **When** the link opens  
    **Then** also fire `external_link_clicked` (existing Phase 1 pattern — see `TrailMapSection`) with an appropriate `link_type` (e.g. `google_maps_place`) + `resort_slug` + `destination_domain`

### G. Pin analytics

13. **Given** a pin or list row that opens the sheet  
    **When** tapped  
    **Then** fire `map_pin_tapped` with:
    - `resort_slug`
    - `place_category` = **chip key** (`restaurant` \| `accommodation` \| `supermarket` \| `convenience` \| `transport` \| `onsen` \| `attraction`) — map `ski_in_ski_out` → `accommodation`
    - `place_source` = `curated` \| `api`  
    **And** update `docs/analytics/tracking-plan.md` + `phase2Events.ts` (`requiredProperties` + instrumentation paths + notes) so `npm run test:analytics` passes

### H. Attribution + failure fallback

14. **Given** the map section  
    **When** rendered  
    **Then** **"Powered by Google"** (or response `attribution` string) is visible as `{typography.caption}`  
    **And** OSM attribution remains correct on the Leaflet control

15. **Given** Places proxy returns `api: []` (failure or empty) while curated may be non-empty  
    **When** the section renders  
    **Then** still show Leaflet map + resort pin + any curated pins/list  
    **And** show a **Google Maps search** fallback link (resort name / area query — not a hard failure empty-state that hides the map)

16. **Given** `GOOGLE_PLACES_API_KEY` is absent server-side  
    **When** the client calls the proxy  
    **Then** UI still works per AC 15 (11.2 already returns empty `api`)

### I. Security

17. **Given** the client bundle  
    **When** inspected  
    **Then** `GOOGLE_PLACES_API_KEY` is **not** present — only `/api/places/nearby` is called from the browser

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** chip↔category mapping (incl. Accommodation → `accommodation` + `ski_in_ski_out`); merge/dedupe helper for multi-category responses; curated-without-coords pin skip; `place_category` mapping for `ski_in_ski_out` → `accommodation`
- [x] **Component / integration tests:** chip toggle / deselect-all; sheet omits rating (logic); curated vs API pin source (logic-level); IntersectionObserver mirrors GettingThereSection (threshold 0.35, once-per-load) — Leaflet not unit-tested
- [x] **Analytics:** update tracking plan + `phase2Events.ts`; `npm run test:analytics` passes; instrumentation files listed for `map_section_viewed` / `map_pin_tapped`
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **E2E:** deferred to Playwright when feasible (CI-4); manual path below for PO
- [x] **Manual QA (PO):** `myoko-kogen` — all chips on; curated restaurant as black sparkle + list entry; API pins grey; sheet no rating; Open in Google Maps; attribution visible; throttle Network to confirm lazy Leaflet load; Places failure still shows resort pin + curated + search link

---

## Tasks / Subtasks

- [x] Add deps: `leaflet`, `react-leaflet@5` (React 19 peer), `@types/leaflet` (AC: C)
- [x] CSP: allow OSM tile host(s) in `web/src/lib/security/csp.ts` (AC: 4, 17)
- [x] Extract `PracticalTipsBlock` from `GettingThereSection` if needed; insert map between Getting There and tips (AC: 1)
- [x] Create `web/src/components/map/` — `ResortMapSection`, Leaflet canvas (dynamic), chips, POI list, POI sheet, Powri-pick sparkle icon (AC: B–H)
- [x] Wire section into `ResortDetailContent` with resort slug + lat/lng + zoom props from detail sections / resort index (AC: 1, 4)
- [x] Client fetch orchestration for multi-chip / Accommodation dual-category merge (AC: 5–7)
- [x] Analytics: tracking plan + `phase2Events.ts` + `track()` calls (AC: 3, 13, 12)
- [x] i18n copy under `resortDetail` (section title, chip labels, sheet CTA, fallback search link, Powri pick) — EN messages
- [x] Tests + verification commands (Testing section)

---

## Dev Notes

### Brownfield (post-11.2)

| Area | Status |
|------|--------|
| `GET /api/places/nearby` | ✅ `{ curated, api, attribution }` |
| `web/src/lib/places/*` | ✅ types, categories, cache, etc. |
| `CuratedMapPlace` | ✅ `placeId`, `category`, `highlight`, `tags`, optional lat/lng — **no `name`** |
| `GooglePlace` | ✅ `placeId`, `name`, `lat`, `lng`, `vicinity`, `primaryType`, optional `photoName` |
| Leaflet / `components/map/` | ✅ Created |
| Map analytics instrumentation | ✅ `ResortMapSection` |
| `ResortDetailContent` | ✅ Map between Getting There and Practical tips |

**Do not reinvent:** reuse `track()` from `@/lib/analytics`, IntersectionObserver pattern from `GettingThereSection`, `external_link_clicked` from `TrailMapSection`, `SectionHeader` for section title styling.

### Architecture compliance

- [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §5.1 CSP · §6.2 `components/map/` · §6.5 dynamic import `ssr: false`
- Response envelope is **11.2’s** `{ curated, api, attribution }` — architecture snippet `{ places, attribution }` is **stale**; do not “fix” the API in 11.3
- Addendum §M section order; §L attribution + no rating

### UX tokens (spines win)

| Token | Value | Use |
|-------|-------|-----|
| `map-section.height` | 240px | Skeleton + map container |
| `rounded.map` | 12px | Map container |
| `map-pin-resort` | accent `#2D6BE4` | Resort center |
| `map-pin-poi` | `#6E6A63` | Google API pins |
| Powri pick | **black sparkle/star** | Curated pins (PO override — hardcoded `#000000`) |
| Motion | map fade-in 200ms; POI sheet 260ms ease-out | `experience.md` |

### Session PO clarifications (2026-07-17)

- Hide map section entirely if resort lat/lng missing
- Fallback Maps search query = exact `nameEn`
- Show search link when any Google-backed category fails or returns empty `api` (partial OK)
- Deselect-all chips → reselect all
- List: API = name + vicinity; curated = “Powri pick”
- Sheet curated empty highlight → badge only (no duplicate text)
- Attribution always hardcoded “Powered by Google”

### Out of scope

- Place Photos `/media` / thumbnails
- Populating curated places for all 20 resorts
- Experience / reviews UI (Epic 12)
- Changing `/api/places/nearby` contract (consume as-is)

---

## Change Log

- 2026-07-17: Implemented Resort Map Section UI (Story 11.3) — Leaflet + chips + POI list/sheet + analytics; PO clarifications applied (hide without lat/lng; exact nameEn search fallback; partial Places empty → search link; deselect-all → select all; curated list/sheet rules).

---

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.5 (dev-story)

### Debug Log References

- PO clarifications 2026-07-17 (9 decisions) captured above.
- Chip count: PO said “6” but enumerated 7 labels — story uses **7**.
- Session clarifications: hide map if no lat/lng; fallback query = exact `nameEn`; show Maps search when any Google-backed category empty/fails; deselect-all → all on; curated list = “Powri pick”; empty-highlight sheet = badge only; attribution hardcoded; branch from `main`.

### Completion Notes List

- Implemented `web/src/components/map/*` with viewport IntersectionObserver → `map_section_viewed` + dynamic Leaflet (`ssr: false`).
- Extracted `PracticalTipsBlock` so order is Getting There → Around the resort → Practical tips.
- Client merge/dedupe in `web/src/lib/map/*`; Accommodation fetches `accommodation` + `ski_in_ski_out`.
- Analytics: `place_source` + chip-key `place_category`; tracking plan + `phase2Events` updated; `test:analytics` green.
- Verified: `lint`, `build`, `test:unit`, `test:analytics`.

### File List

- `_powri/implementation-artifacts/Epic-11/11-3-resort-map-section-ui.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `docs/analytics/tracking-plan.md`
- `web/package.json`
- `web/package-lock.json`
- `web/messages/en.json`
- `web/src/app/globals.css`
- `web/src/lib/security/csp.ts`
- `web/src/lib/analytics/phase2Events.ts`
- `web/src/lib/resort/detailSections.ts`
- `web/src/lib/map/chips.ts`
- `web/src/lib/map/chips.test.ts`
- `web/src/lib/map/types.ts`
- `web/src/lib/map/mergePlaces.ts`
- `web/src/lib/map/mergePlaces.test.ts`
- `web/src/lib/map/googleMapsUrls.ts`
- `web/src/lib/map/googleMapsUrls.test.ts`
- `web/src/lib/map/fetchNearby.ts`
- `web/src/lib/map/fetchNearby.test.ts`
- `web/src/lib/map/poiSheetDisplay.ts`
- `web/src/lib/map/poiSheetDisplay.test.ts`
- `web/src/lib/map/pinIcons.test.ts`
- `web/src/components/map/ResortMapSection.tsx`
- `web/src/components/map/ResortLeafletMap.tsx`
- `web/src/components/map/MapSkeleton.tsx`
- `web/src/components/map/CategoryChips.tsx`
- `web/src/components/map/PoiList.tsx`
- `web/src/components/map/PoiSheet.tsx`
- `web/src/components/map/mapIcons.ts`
- `web/src/components/resort/ResortDetailContent.tsx`
- `web/src/components/resort/GettingThereSection.tsx`