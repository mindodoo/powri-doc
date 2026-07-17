baseline_commit: ef2d7ef

# Story 11.5: Mapbox Migration (replace Leaflet/OSM)

Status: done

<!-- Follow-up to Story 11.3 — PO wants Mapbox for closer brand-DNA control over map styling. Rendering-engine swap ONLY; no UX/behavior change (that's 11.6-11.9). -->

## Story

As **Powri**,
I want the resort map rendered with Mapbox instead of Leaflet/OSM,
so that the map's visual styling can be customized closer to our brand instead of using generic OSM tiles.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) · [`architecture/phase2/architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §Maps — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.3** merged to `main`. Story **11.4** (bug fixes) recommended first — touches the same `PoiSheet.tsx`/`googleMapsUrls.ts` files; land 11.4 before rebasing this story to avoid rework.

**Blocks:** Stories **11.6** (pin shapes), **11.7** (chip model/Ski Resort), **11.8** (radius/fit-bounds), **11.9** (split-view) — all build on the Mapbox primitives this story introduces. Do these AFTER this story merges.

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Integration approach | `react-map-gl` (thin React wrapper), not raw `mapbox-gl-js` |
| Map style | Mapbox's standard **Light** style (`mapbox://styles/mapbox/light-v11`) — no custom Studio style yet |
| Access token | PO already has a Mapbox account/token |
| Scope | **Rendering engine swap only.** Pin shapes, chip model, radius/cap, and desktop layout are separate stories — do not fold that work in here |
| Map zoom (2026-07-17 PO) | **Free scroll-wheel + trackpad pinch** on the map (`scrollZoom` enabled). No +/- `NavigationControl`. Accept embedded-map scroll-trap risk on desktop. `cooperativeGestures` not used — it requires Ctrl/Cmd+scroll and conflicts with free wheel zoom |

---

## Root-cause context (why this also fixes a latent bug)

Leaflet's own CSS sets internal z-indices (`leaflet-marker-pane: 600`, `leaflet-popup-pane: 700`, `leaflet-control: 800`, `leaflet-top/bottom: 1000`) that are **not contained** to `.leaflet-container` (it has `position: relative` but no `z-index`, so it never forms its own stacking context — these values compete directly against the rest of the app's z-scale, which tops out at `z-[70]` outside full-screen flows). This is why `PoiSheet.tsx`'s desktop-centered modal (`z-[60]`/`z-[61]`) can render underneath map internals. Give the new Mapbox container its own isolated stacking context from day one so this class of bug can't recur (see AC 5) — Story 11.9 owns the bigger desktop layout change, but the base isolation belongs here since this story already rewrites the container from scratch.

---

## Acceptance Criteria

### A. Dependencies

1. **Given** `web/package.json`
   **When** this story ships
   **Then** `leaflet`, `react-leaflet`, `@types/leaflet` are removed and `mapbox-gl` (`^3.5` or later — required for the `react-map-gl/mapbox` sub-import) + `react-map-gl` (`^8.1`) are added
   **And** `npm install` succeeds without `--legacy-peer-deps` (react-map-gl's peer range is `react>=16.3`; verify actual install against this repo's React `19.2.7` — if peer resolution fails, document the workaround used)

2. **Given** `NEXT_PUBLIC_MAPBOX_TOKEN`
   **When** added to `web/.env.local` / `web/.env.example`
   **Then** it is documented as a **public, client-exposed** token by design (unlike `GOOGLE_PLACES_API_KEY` — Mapbox access tokens are meant for client-side use and are scoped/restricted via the Mapbox account's URL restrictions, not secrecy)

### B. Map component rewrite

3. **Given** `web/src/components/map/ResortLeafletMap.tsx`
   **When** migrated
   **Then** it is replaced by `web/src/components/map/ResortMapboxMap.tsx` using `<Map>` from `react-map-gl/mapbox`, `mapboxAccessToken={NEXT_PUBLIC_MAPBOX_TOKEN}`, `mapStyle="mapbox://styles/mapbox/light-v11"`, centered on `{latitude, longitude}` at `zoom={mapDefaultZoom}` (same prop contract as the Leaflet version it replaces)

4. **Given** `ResortMapSection.tsx`'s dynamic import
   **When** updated
   **Then** it imports `ResortMapboxMap` instead of `ResortLeafletMap` (still `next/dynamic`, `ssr: false`, same `MapSkeleton` loading fallback — NFR-16 lazy-load behavior unchanged)

5. **Given** the map container
   **When** rendered
   **Then** it has `isolate` (CSS `isolation: isolate`) on its wrapper, establishing a new stacking context so Mapbox's internal DOM (its own controls/markers, which also carry z-index values) can never outrank page-level overlays like `PoiSheet` — defensive fix independent of Story 11.9's larger layout work

6. **Given** `web/src/components/map/mapIcons.ts`
   **When** rewritten for `react-map-gl`'s `<Marker>` (which takes React children, not Leaflet `DivIcon` HTML strings)
   **Then** resort-center, API, and curated pins render as **the exact same visuals as today** (blue dot, grey dot, black sparkle) — this story does NOT change pin shapes (that's 11.6); it only ports the existing markup from `L.divIcon({ html })` strings to JSX

7. **Given** pin tap interaction
   **When** a `<Marker>` is clicked
   **Then** `onPlaceSelect(place)` fires identically to the current `eventHandlers.click` on Leaflet's `<Marker>` — no behavior change

8. **Given** the map canvas
   **When** the visitor scrolls the wheel or pinch-zooms on the map (desktop trackpad or touch)
   **Then** zoom in/out works (Mapbox `scrollZoom` + default `touchZoomRotate`; no `NavigationControl` +/- buttons)
   **And** PO accepts embedded-map scroll-trap risk when the pointer is over the 240px map on desktop

### C. CSP + architecture doc

9. **Given** `web/src/lib/security/csp.ts`
   **When** updated
   **Then**:
   - Remove `*.tile.openstreetmap.org` from `img-src`
   - Add Mapbox's tile/style/sprite/glyph hosts to `img-src`/`connect-src` as needed (`https://api.mapbox.com`, `https://events.mapbox.com` for telemetry — verify exact hosts against Mapbox GL JS's network requests during manual QA, since Mapbox uses multiple subdomains for tiles/fonts/sprites)
   - Add `worker-src 'self' blob:` (or extend existing `script-src`) — `mapbox-gl-js` spins up a Web Worker via a `blob:` URL for tile parsing; without this the map fails silently under CSP

10. **Given** `architecture-phase2.md` FR-9 row (line ~39) and §Maps table (line ~60) and the CSP note (line ~230, ~399)
   **When** updated
   **Then** they read "Mapbox + `react-map-gl`" instead of "Leaflet + OSM" / "Leaflet + react-leaflet" — this is a live architecture decision change, not just a code change

### D. Parity — nothing else regresses

11. **Given** everything downstream of the map canvas (chips, POI list/sheet, fallback search link, attribution caption, `map_section_viewed`/`map_pin_tapped` analytics, IntersectionObserver lazy-load)
    **When** this story ships
    **Then** all of it is **unchanged** — this story only swaps what's inside the 240px map container

12. **Given** Mapbox's own required attribution (logo + "© Mapbox © OpenStreetMap" link, per Mapbox ToS)
    **When** the map renders
    **Then** Mapbox's attribution control is present (default `react-map-gl` behavior — do not disable it) **in addition to** the existing "Powered by Google" caption below the map (two separate, non-conflicting attributions for two separate data sources)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `pinIcons.test.ts` still passes (it tests `placesWithCoordinates`/source distinction, not Leaflet APIs directly — verify no accidental Leaflet import remains anywhere in `src/lib/map/**`)
- [x] **Build:** `npm run build` — confirms no leftover `leaflet`/`react-leaflet` imports break the build
- [x] **Lint / unit:** `npm run lint && npm run test:unit`
- [x] **Manual QA (PO):** `myoko-kogen` — map renders with Light style, resort pin + curated sparkle + API dots visible, pan/zoom works, "Open in Google Maps"/list/sheet still work, throttle Network to confirm lazy-load still defers Mapbox bundle until viewport entry, verify CSP doesn't block tiles/worker in browser console

---

## Tasks / Subtasks

- [x] `web/package.json`: remove `leaflet`/`react-leaflet`/`@types/leaflet`, add `mapbox-gl`/`react-map-gl` (AC: 1)
- [x] `web/.env.local` + `.env.example`: `NEXT_PUBLIC_MAPBOX_TOKEN` (AC: 2)
- [x] Create `web/src/components/map/ResortMapboxMap.tsx`, delete `ResortLeafletMap.tsx` (AC: 3, 6, 7)
- [x] `web/src/components/map/ResortMapSection.tsx`: update dynamic import + wrapper `isolate` class (AC: 4, 5)
- [x] `web/src/components/map/mapIcons.tsx`: port DivIcon HTML → JSX marker content (AC: 6)
- [x] `web/src/lib/security/csp.ts`: swap OSM → Mapbox hosts + `worker-src blob:` (AC: 8)
- [x] `_powri/planning-artifacts/architecture/phase2/architecture-phase2.md`: update FR-9/§Maps/CSP rows (AC: 9)
- [x] Mapbox attribution enabled by default in `ResortMapboxMap` (no `attributionControl={false}`) — PO manual browser verify pending (AC: 11)
- [x] Enable map zoom gestures — free scroll wheel + trackpad pinch; no NavigationControl (AC: 8; PO amend 2026-07-17)
- [x] Run full verification commands

---

## Dev Notes

### Brownfield — files this story modifies

| File | Current state | This story's change |
|------|---------------|----------------------|
| `web/src/components/map/ResortLeafletMap.tsx` | `MapContainer`/`TileLayer`/`Marker` from `react-leaflet`, OSM tile URL | Deleted, replaced by `ResortMapboxMap.tsx` |
| `web/src/components/map/ResortMapSection.tsx:30-33` | `dynamic(() => import('./ResortLeafletMap'), { ssr: false, loading: () => <MapSkeleton /> })` | Import path only changes; dynamic/ssr:false/skeleton pattern is identical |
| `web/src/components/map/mapIcons.ts` | `L.divIcon({ html: '<span>...</span>' })` × 3 (resort/API/curated) | Same 3 visual outputs, as `react-map-gl` `<Marker>` children (React nodes) instead of Leaflet `DivIcon` |
| `web/src/lib/security/csp.ts:39` | `img-src` allows `https://*.tile.openstreetmap.org` | Mapbox hosts + worker-src |
| `architecture-phase2.md:39,60,230,399` | Documents Leaflet+OSM as the FR-9 decision | Mapbox + react-map-gl |

### Latest library specifics (verified 2026-07-17)

- `react-map-gl` latest stable: **8.1.1** (Apr 2026). v8+ split into subpath imports — use **`react-map-gl/mapbox`** (for `mapbox-gl>=3.5.0`), not the legacy default export or `react-map-gl/maplibre`.
- React 19 compatibility: not yet reflected in the package's peer-dependency range (`react>=16.3`), but confirmed working in practice per the maintainers' GitHub discussions. This repo runs React `19.2.7` — install normally first; only reach for `--legacy-peer-deps` if npm's resolver actually blocks it, and note that in Dev Agent Record if used.
- `mapbox-gl-js` requires a Web Worker (via `blob:`) for tile decoding — this is the most common CSP gotcha when migrating; test in an actual browser (not just build success) before marking done.

### Architecture compliance

- [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) — this story is itself the architecture update for the Maps row; update the doc, don't just diverge from it.
- Keep the existing `next/dynamic({ ssr: false })` pattern — Mapbox GL JS also requires `window`, same constraint as Leaflet.

### Out of scope

- Pin shapes per category — Story 11.6
- Chip model, "All" chip, Ski Resort category — Story 11.7
- Search radius / pin density / auto-fit-bounds — Story 11.8
- Desktop split-view layout, list simplification — Story 11.9
- Custom Mapbox Studio style — deferred until PO wants brand-specific styling beyond stock Light

---

## Change Log

- 2026-07-17: Story created — PO follow-up request to move off Leaflet/OSM for brand-DNA flexibility.
- 2026-07-17: PO amend — enable free scroll-wheel/trackpad pinch zoom (removed `scrollZoom={false}`); no +/- controls; accept desktop scroll-trap risk.
- 2026-07-17: PO manual QA pass on `myoko-kogen` — story marked done.

---

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm install` succeeded without `--legacy-peer-deps` on React 19.2.7.
- `react-map-gl` v8 `<Map>` does not accept `className`; used `style={{ width/height: '100%' }}` inside the rounded/isolated parent wrapper.

### Completion Notes List

- Replaced Leaflet/OSM with Mapbox Light (`mapbox://styles/mapbox/light-v11`) via `react-map-gl/mapbox`.
- Pin visuals ported to JSX components in `mapIcons.tsx` (renamed from `.ts` for JSX).
- Map wrapper uses `isolate` for stacking-context isolation (AC 5).
- CSP: removed OSM tile host; added Mapbox `connect-src`/`img-src` hosts + `worker-src blob:`.
- **Deploy reminder:** add `NEXT_PUBLIC_MAPBOX_TOKEN` to Vercel Preview + Production (Root Directory `web`); restrict token URLs in Mapbox account.
- PO amend: removed `scrollZoom={false}` so wheel + trackpad pinch zoom in/out; no `NavigationControl`. Mapbox `cooperativeGestures` not set — it forces Ctrl/Cmd+scroll and conflicts with free wheel zoom.
- PO manual QA pass on `myoko-kogen` (map render, pins, zoom in/out, CSP/worker, lazy-load, dual attribution).

### File List

- `web/package.json` (modified)
- `web/package-lock.json` (modified)
- `web/.env.example` (modified)
- `web/src/components/map/ResortMapboxMap.tsx` (added)
- `web/src/components/map/ResortLeafletMap.tsx` (deleted)
- `web/src/components/map/ResortMapSection.tsx` (modified)
- `web/src/components/map/mapIcons.tsx` (added — replaces `mapIcons.ts`)
- `web/src/components/map/mapIcons.ts` (deleted)
- `web/src/lib/security/csp.ts` (modified)
- `web/src/app/globals.css` (modified — comment only)
- `web/src/lib/map/pinIcons.test.ts` (modified — test description)
- `_powri/planning-artifacts/architecture/phase2/architecture-phase2.md` (modified)
- `_powri/implementation-artifacts/Epic-11/11-5-mapbox-migration.md` (modified)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
