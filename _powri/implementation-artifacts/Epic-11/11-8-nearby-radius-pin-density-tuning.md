baseline_commit: ef2d7ef

# Story 11.8: Nearby Radius & Pin Density Tuning

Status: done

<!-- Follow-up to Story 11.3/11.2 — pins feel "too far" due to viewport clipping, not an API bug. Tightens radius/cap and adds auto-fit-bounds. Depends on Mapbox migration (11.5). -->

## Story

As a **visitor browsing the resort map**,
I want nearby places to feel close to the resort and the map to always show the pins that are active,
so that the map doesn't feel sparse/overwhelming and I don't have to pan to find results I already filtered for.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration) — auto-fit-bounds uses `react-map-gl`'s map ref API. Benefits from Story **11.7** (single-select chip model) since with that model, the "everything at once" case only happens under "All," simplifying the bounds math.

**Amends:** Story 11.2's PO decision (5 km radius / max 10 results per category) — recorded here, not by editing 11.2's file retroactively.

---

## Root-cause investigation (2026-07-17)

The 5 km radius is **already correctly applied** server-side (`NEARBY_SEARCH_RADIUS_METERS = 5000` in `web/src/lib/places/searchNearby.ts`) — this is not a bug. The "too far" perception is **viewport clipping**:

- Map container is fixed at **240px tall** (`{components.map-section.height}` token) at default **zoom 13**.
- Web Mercator gives ≈15 m/px at zoom 13 for Japan's latitude range (~36-43°N, `cos(lat) ≈ 0.8`).
- Half the container height (120px) ≈ **~1.8 km** — any pin more than ~1.8 km north/south of the resort center is already outside the visible strip, even though it's well inside the 5 km search radius. Only the wider (horizontal) axis has room for ~4.6 km before clipping.

**PO decision (2026-07-17):** fix both ends — tighten the actual search radius/cap **and** make the map auto-fit to whatever pins are currently active, so nothing legitimately-in-range ever renders off-frame regardless of container size.

---

## Acceptance Criteria

### A. Server-side radius/cap tuning

1. **Given** `web/src/lib/places/searchNearby.ts`
   **When** updated
   **Then** `NEARBY_SEARCH_RADIUS_METERS` reduces from `5000` to **`3000`** (3 km) — chosen to better match what a small, fixed-height map can visually represent while still covering realistic walking/short-drive distance from a resort base
   **And** `NEARBY_SEARCH_MAX_RESULTS` reduces from `10` to **`8`** (per-category cap, unchanged mechanism — just a lower constant)

2. **Given** `web/src/lib/places/searchNearby.test.ts` (or equivalent)
   **When** updated
   **Then** assertions on the request body reflect the new radius/cap constants

3. **Given** cached `places_cache` rows from before this change (5 km/max 10 payloads)
   **When** their TTL (`PLACES_CACHE_TTL_HOURS`, default 168h) expires
   **Then** they're naturally refetched under the new radius/cap — no manual cache invalidation required (note this in Dev Agent Record so Ops isn't surprised results shrink gradually rather than instantly)

### B. Map auto-fit-bounds

4. **Given** the active pin set changes (chip selection changes, or initial load)
   **When** the map (re)renders
   **Then** it calls `fitBounds()` (via `react-map-gl`'s map ref) over the resort center **plus** all currently-visible pins with reasonable padding, instead of always sitting at a fixed `mapDefaultZoom`

5. **Given** the active category/selection has **zero** pins with coordinates
   **When** auto-fit runs
   **Then** it falls back to centering on the resort at `mapDefaultZoom` (must not throw/no-op badly on empty bounds — guard explicitly)

6. **Given** only the resort center pin exists (no API/curated pins yet, e.g. still loading)
   **When** auto-fit runs
   **Then** it shows the resort at `mapDefaultZoom` (same fallback as AC 5) rather than an extreme zoom-to-a-single-point

7. **Given** auto-fit is recalculated on every selection change
   **When** implemented
   **Then** it's debounced/guarded against redundant calls during the existing 150ms fetch-debounce window in `ResortMapSection.tsx` (don't fight the existing fetch debounce with a competing animation on every keystroke-like state change)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** radius/cap constants test; a pure bounds-calculation helper (extract `computeBounds(places, resortCenter)` into `web/src/lib/map/` so it's unit-testable without mounting Mapbox) covering: normal case, zero-pins fallback, single-point fallback
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA (PO):** `myoko-kogen` — switch between categories, confirm the map re-frames to show all active pins without manual panning; confirm empty-category and loading states don't zoom oddly

---

## Tasks / Subtasks

- [x] `web/src/lib/places/searchNearby.ts`: `NEARBY_SEARCH_RADIUS_METERS = 3000`, `NEARBY_SEARCH_MAX_RESULTS = 8` (AC: 1)
- [x] Update associated unit tests for new constants (AC: 2)
- [x] Extract `computeBounds()` helper in `web/src/lib/map/` (pure function: places + resort center → bounds or null) (AC: 4-6)
- [x] `web/src/components/map/ResortMapboxMap.tsx`: call `fitBounds()` via map ref when the pin set changes, using `computeBounds()` (AC: 4-7)
- [x] Run full verification commands

---

## Dev Notes

### Brownfield

| File | Current state | This story's change |
|------|----------------|----------------------|
| `web/src/lib/places/searchNearby.ts:6-7` | `NEARBY_SEARCH_RADIUS_METERS = 5000`, `NEARBY_SEARCH_MAX_RESULTS = 10` | `3000` / `8` |
| `web/src/components/map/ResortMapboxMap.tsx` (post-11.5) | Static `center`/`zoom` props, never reframes after mount | Adds `fitBounds()` on pin-set change via map ref |
| `web/src/components/map/ResortMapSection.tsx:86-110` | 150ms debounced fetch effect on `selectedChips` change | Bounds recompute should key off the same `places`/`pinPlaces` derived state (`useMemo` at line 112), not duplicate the debounce logic |

### Why this doesn't touch 11.2's file

11.2's story record documents the 5 km/max-10 decision as it stood at the time — accurate at merge, now superseded. Per this epic's established pattern (11.4 also amends-not-edits 11.3), leave 11.2's file as a historical record and let this story's own file be the source of truth for the current values.

### Out of scope

- Mapbox migration itself — Story 11.5 (prerequisite)
- Pin shapes — Story 11.6
- Chip model / Ski Resort — Story 11.7
- Desktop split-view / list changes — Story 11.9

---

## Change Log

- 2026-07-17: Story created — PO follow-up request after diagnosing viewport-clipping root cause for "pins feel too far."
- 2026-07-17: Implemented — radius 3 km / cap 8, `computeBounds` helper, auto-fit-bounds (40px padding, 300ms animation, maxZoom = mapDefaultZoom); immediate reframe on chip selection + reframe when `pinPlaces` updates.
- 2026-07-17: PO manual QA passed on `myoko-kogen` — story marked done.

---

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- PO clarifications: 40px padding; immediate reframe on selection then on pinPlaces update; animated fitBounds with maxZoom = mapDefaultZoom; new `searchNearby.test.ts`; branch `feature/11-8-nearby-radius-pin-density-tuning` from merged main.

### Completion Notes List

- AC 1–2: `NEARBY_SEARCH_RADIUS_METERS` → 3000, `NEARBY_SEARCH_MAX_RESULTS` → 8; new `searchNearby.test.ts` asserts request body.
- AC 3: No cache invalidation — existing `places_cache` rows refetch naturally on TTL expiry (default 168h).
- AC 4–7: `computeBounds()` returns `[[west,south],[east,north]]` or `null` (resort-only fallback). `ResortMapboxMap` calls `fitBounds` / `flyTo` via map ref on `selectionKey` + `places` change; redundant identical fits skipped via fit-key guard.
- Verification: `npm run lint`, `npm run build`, `npm run test:unit` — all pass (266 tests).
- PO manual QA on `myoko-kogen` confirmed category switching reframes correctly; empty/loading states behave as expected.

### File List

- `web/src/lib/places/searchNearby.ts`
- `web/src/lib/places/searchNearby.test.ts` (new)
- `web/src/lib/map/computeBounds.ts` (new)
- `web/src/lib/map/computeBounds.test.ts` (new)
- `web/src/components/map/ResortMapboxMap.tsx`
- `web/src/components/map/ResortMapSection.tsx`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `_powri/implementation-artifacts/Epic-11/11-8-nearby-radius-pin-density-tuning.md`
