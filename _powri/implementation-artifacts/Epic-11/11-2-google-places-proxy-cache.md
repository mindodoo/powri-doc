baseline_commit: 185cdeaea51888d1cf3ec30468348d835de5fdba

# Story 11.2: Google Places Proxy & Cache

Status: done

<!-- PO decisions 2026-07-16: resortSlug, {curated,api,attribution}, curated lat/lng in content, 5km/10 results, GCP alert = PO manual DoD -->
<!-- PO decisions 2026-07-16 (photos): places.photos in field mask + cache photoName metadata; no Place Photos /media; no thumbnail Phase 2 -->

## Story

As the **system**,  
I want Places API calls server-side with caching, merged with any curated editorial entries,  
So that POI data stays secure, cost-bounded, and surfaces PO-curated picks.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.1** (geo + `curatedMapPlaces` on resort index) · Epic **9.1** (`places_cache` table + `createAdminClient()`)

**Blocks:** Story **11.3** (map UI consumes this route)

**Reference only (not source of truth):** [`input_docs/story-curated-map-places.md`](../../../input_docs/story-curated-map-places.md)

---

## PO decisions (2026-07-16)

| Topic | Decision |
|-------|----------|
| Query param | `resortSlug` (camelCase) |
| Response envelope | `{ curated, api, attribution }` |
| Curated pin coordinates | **Optional `latitude` / `longitude` on each `curated_map_places` entry** in content — PO supplies at edit time; no Place Details API in 11.2 |
| Nearby Search | **5 km** radius, **max 10** results |
| Photos | **`places.photos` in Nearby Search field mask** (Pro tier, $0 marginal) — cache first photo `name` as optional `photoName` on `GooglePlace`. **No** Place Photos `/media` calls in 11.2. **No thumbnail in Phase 2 UI** (11.3) — metadata cached for a future phase |
| GCP billing alert | PO/Ops **manual DoD** checklist item (not code) |
| Start gate | After 11.1 on `main` — pull latest before dev |

---

## Acceptance Criteria

### A. Curated schema extension (PO decision — coordinates in content)

1. **Given** `content/content-model.md` §4.1a `curated_map_places` subsection  
   **When** an editor reads the doc  
   **Then** optional per-entry `latitude` (−90…90) and `longitude` (−180…180) are documented for map pin placement

2. **Given** `curatedMapPlaceSchema` in `schema.ts` and `CuratedMapPlace` in `types.ts`  
   **When** frontmatter is parsed  
   **Then** optional `latitude` / `longitude` map to camelCase `latitude` / `longitude` on `CuratedMapPlace`  
   **And** omission is valid (entry returned in `curated` array but 11.3 skips map pin when coords missing)

3. **Given** PO pilot `myoko-kogen` curated entry  
   **When** reviewed  
   **Then** PO adds valid `latitude` / `longitude` for the pilot restaurant *(manual PO task in implementation)*

### B. Route, security, and resort index

4. **Given** `GOOGLE_PLACES_API_KEY` is server-only  
   **When** the route handler runs  
   **Then** the key is read from `process.env` only on the server

5. **Given** `GET /api/places/nearby?resortSlug=niseko-united&category=restaurant`  
   **When** lat/lng are needed for the Google call  
   **Then** resort center coordinates come from `getResortBySlug(resortSlug)` — **not** from client params

6. **Given** an unknown `resortSlug`  
   **When** the route is called  
   **Then** return `404` / `NOT_FOUND`

7. **Given** a missing or invalid `category`  
   **When** the route is called  
   **Then** return `422` / `VALIDATION_ERROR`

### C. Response shape — curated + API merge

8. **Given** a successful request  
   **When** the JSON is returned  
   **Then** shape is:

   ```typescript
   {
     curated: CuratedMapPlace[];  // from build-time index, filtered by category
     api: GooglePlace[];          // cached Google results, deduped
     attribution: string;         // e.g. "Powered by Google"
   }
   ```

9. **Given** curated entries for the requested category  
   **When** the response is built  
   **Then** `curated` is sourced from the resort index (no DB call)

10. **Given** API results overlap curated `placeId`s  
    **When** the response is built  
    **Then** duplicates are removed from `api` (curated wins)

11. **Given** `category=ski_in_ski_out`  
    **When** the route is called  
    **Then** no Google Nearby Search call  
    **And** `{ curated: [...], api: [], attribution }`

12. **Given** Google Places API fails  
    **When** the route is called  
    **Then** return **200** with `api: []`, curated still present, `attribution` included

### D. Google Places — Pro-tier field mask

13. **Given** a non–`ski_in_ski_out` category with cache miss  
    **When** the server calls Places API (New) `searchNearby`  
    **Then** `POST https://places.googleapis.com/v1/places:searchNearby` with:
    - `locationRestriction.circle`: center = resort lat/lng, **radius 5000 m**
    - `maxResultCount`: **10**
    - `includedTypes` from category map (see Dev Notes)
    - `X-Goog-FieldMask`: `places.id,places.displayName,places.shortFormattedAddress,places.location,places.primaryType,places.photos`
    - **Never** `places.rating`, `places.userRatingCount`, or other Enterprise-tier fields
    - **Never** call Place Photos `/media` — metadata only (see PO photos decision)

14. **Given** a normalized `GooglePlace`  
    **When** returned in `api`  
    **Then** required fields are: `placeId`, `name`, `lat`, `lng`, `vicinity`, `primaryType` — **no `rating`**  
    **And** when Google returns `photos[]`, include optional `photoName` (first element's `name` resource string)  
    **And** when no photos, omit `photoName` (do not send `null`)

15. **Given** any request to this route  
    **When** the handler runs  
    **Then** it **never** calls `places.googleapis.com/v1/.../media` (Place Photos SKU — Enterprise, 1,000 free/mo; out of scope)

### E. `places_cache`

16. **Given** cache miss or stale `(resort_slug, category)`  
    **When** Google call succeeds  
    **Then** upsert `places_cache` via `createAdminClient()` — `payload` = normalized `api` array (incl. `photoName` when present), `fetched_at` = now

17. **Given** cache hit within TTL  
    **When** the route is called  
    **Then** serve cached `api` without calling Google; merge fresh `curated` from index

18. **Given** `PLACES_CACHE_TTL_HOURS`  
    **When** TTL is evaluated  
    **Then** default **168**, minimum **24** (clamp below 24 → 24; invalid → 168)

### F. Rate limiting

19. **Given** policy `places` at **60/min/IP**  
    **When** exceeded on `/api/places/nearby`  
    **Then** `429` / `RATE_LIMIT` + `Retry-After`

### G. Ops (manual — PO/Ops)

20. **Given** GCP Places API project  
    **When** story is marked done  
    **Then** PO confirms billing budget alert is configured *(checklist below)*

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `web/src/lib/places/**` — category map, TTL parser, dedup, field mask, normalizer
- [x] **Unit (Vitest):** curated lat/lng optional parsing in `schema.test.ts` (extend 11.1 tests)
- [x] **Unit (Vitest):** `places` rate-limit policy in `limiter.test.ts`
- [x] **Route tests:** cache hit/miss, API failure → empty `api`, `ski_in_ski_out` no-call, dedup, 404/422
- [x] **Analytics:** N/A (11.3)
- [x] **Content:** update pilot `myoko-kogen` curated entry with lat/lng; `npm run test:launch` passes
- [x] **Build / lint / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA:** real key — `myoko-kogen` + `restaurant`; curated has coords; no `rating`; `photoName` present when Google returns photos; no `/media` calls; cache row in Supabase
- [x] **PO/Ops manual:** GCP billing budget alert confirmed

---

## Tasks / Subtasks

- [x] Extend `CuratedMapPlace` + `curatedMapPlaceSchema` with optional `latitude`/`longitude` (AC: 1–2)
- [x] Update `content/content-model.md` §4.1a + `_TEMPLATE.md` commented example (AC: 1)
- [x] PO: add lat/lng to `myoko-kogen` pilot curated entry (AC: 3)
- [x] Add `PLACES_CACHE_TTL_HOURS` to `web/.env.example` (AC: 18)
- [x] Create `web/src/lib/places/categories.ts` (AC: 13)
- [x] Create `web/src/lib/places/types.ts` — `GooglePlace`, `NearbyPlacesResponse` (AC: 8, 14)
- [x] Create `web/src/lib/places/fieldMask.ts` + test — includes `places.photos`; asserts no Enterprise fields (AC: 13–15)
- [x] Create `web/src/lib/places/normalize.ts` — map first `photos[0].name` → `photoName?` (AC: 14)
- [x] Create `web/src/lib/places/cache.ts` (AC: 16–18)
- [x] Create `web/src/lib/places/nearby.ts` — orchestration (AC: 5–12)
- [x] Create `web/src/app/api/places/nearby/route.ts` (AC: 4–12)
- [x] Add `places` to `RATE_LIMIT_POLICIES` + `middleware.ts` (AC: 19)
- [x] Tests (see Testing section)
- [x] PO/Ops: GCP billing alert (AC: 20)
- [x] Run full verification commands

---

## Dev Notes

### Brownfield (post-11.1)

| Area | Status |
|------|--------|
| `curatedMapPlaces` on `Resort` | ✅ 11.1 — no lat/lng on curated yet (**extend in this story**) |
| `getResortBySlug()` | ✅ `web/src/lib/content/index.ts` |
| `places_cache` | ✅ `supabase/migrations/001_phase2_core.sql` |
| `createAdminClient()` | ✅ `web/src/lib/supabase/admin.ts` |
| `/api/places/nearby` | ❌ Create |
| `web/src/lib/places/` | ❌ Create |
| `places` rate limit | ❌ Add to `limiter.ts` + `middleware.ts` |

**Branch note:** Dev should run from `main` with 11.1 merged. Verify `CuratedMapPlace` exists after `git pull`.

### Curated coordinate rule (PO decision)

- Coords live in **content**, not fetched at runtime.
- Route returns all curated entries for the category; entries without lat/lng are valid (11.3 renders list/sheet but omits pin).
- `ski_in_ski_out` relies entirely on PO-supplied coords — no API fallback.

### Category → Google `includedTypes`

| Category | `includedTypes` | API call? |
|----------|-----------------|-----------|
| `ski_in_ski_out` | — | **Never** |
| `accommodation` | `lodging` | Yes |
| `restaurant` | `restaurant` | Yes |
| `supermarket` | `supermarket` | Yes |
| `convenience` | `convenience_store` | Yes |
| `transport` | `bus_station`, `transit_station`, `train_station` | Yes — single request with multiple types |
| `onsen` | `spa` | Yes |
| `attraction` | `tourist_attraction` | Yes |

Verify type strings against [Places API types](https://developers.google.com/maps/documentation/places/web-service/place-types) at implementation time.

### Cache flow

```text
validate resortSlug + category
→ getResortBySlug → resort lat/lng + curatedMapPlaces
→ curated = filter by category
→ if ski_in_ski_out → return { curated, api: [], attribution }
→ read places_cache (service role); if fresh → api = payload
→ else searchNearby (5km, max 10) → normalize → upsert cache
→ on Google error → api = []
→ dedup api against curated placeIds
→ return { curated, api, attribution: "Powered by Google" }
```

Cache stores **API results only**. Curated always from build-time index.

### Photos — metadata only (PO decision 2026-07-16)

| Layer | Action | SKU | Cost |
|-------|--------|-----|------|
| Nearby Search field mask | Include `places.photos` | Nearby Search Pro | $0 marginal (same billable event) |
| Normalized response | Optional `photoName` from `photos[0].name` | — | — |
| Place Photos `/media` | **Out of scope** 11.2 + Phase 2 UI | Place Details Photos (Enterprise) | Would consume 1,000 free/mo cap |
| 11.3 POI sheet | **No thumbnail** Phase 2 | — | `photoName` ignored by UI for now |

### `GooglePlace` type (11.3 contract)

```typescript
interface GooglePlace {
  placeId: string;
  name: string;
  lat: number;
  lng: number;
  vicinity: string;      // from shortFormattedAddress
  primaryType: string;
  photoName?: string;    // e.g. "places/ChIJ…/photos/AUacShh…" — cached, not displayed Phase 2
}
```

### Files to touch

| File | Change |
|------|--------|
| `content/content-model.md` | Optional curated lat/lng |
| `content/resorts/_TEMPLATE.md` | Comment example with lat/lng |
| `content/resorts/myoko-kogen.md` | PO pilot coords |
| `web/src/lib/content/schema.ts` | Optional lat/lng on curated schema |
| `web/src/lib/content/types.ts` | `CuratedMapPlace.latitude?`, `.longitude?` |
| `web/src/lib/content/schema.test.ts` | Optional coords cases |
| `web/src/lib/places/*` | **New** module |
| `web/src/app/api/places/nearby/route.ts` | **New** |
| `web/src/lib/rate-limit/limiter.ts` | `places` policy |
| `web/src/middleware.ts` | Rate-limit route |
| `web/.env.example` | `PLACES_CACHE_TTL_HOURS` |

### Out of scope

- Leaflet UI, CSP, analytics (11.3)
- POI thumbnails / Place Photos `/media` / `/api/places/photo` proxy (Phase 2 — no thumbnail per PO)
- `place_source` analytics property (defer 11.3 / Epic 15)
- Populating curated data for all 20 resorts

### Architecture compliance

- [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §5, §5.1, §10
- [`addendum.md`](../../planning-artifacts/prds/phase2/addendum.md) §F, §L
- Service role for `places_cache` only — `createAdminClient()` never in client bundle

### Cost note

Nearby Search Pro only (no Place Details, no Place Photos `/media`). Field mask incl. `places.photos` does not add billable events. 5 km / 10 results — stays within 5,000/mo Pro free cap per addendum §F. `photoName` cached for a future phase without triggering the 1,000/mo Place Details Photos cap.

---

## Dev Agent Record

### Agent Model Used

Composer (dev-story — 2026-07-16)

### Debug Log References

- PO clarifications (2026-07-16): branch from `main`; dev adds myoko coords; missing resort geo → 422; missing API key → 200 empty `api`; missing Supabase service role → 503.

### Completion Notes List

- Added optional `latitude`/`longitude` on curated map places (schema, types, content-model, template, myoko pilot).
- Pilot coords `36.8925536, 138.1773094` sourced via OSM cross-reference for Akakura Restaurant Shibata (no local Google API key for Place Details lookup).
- New `web/src/lib/places/` module: category map, Pro-tier field mask, normalizer (`photoName` optional), TTL parser, Supabase cache, Nearby Search client, orchestration with curated dedup.
- `GET /api/places/nearby?resortSlug=&category=` returns `{ curated, api, attribution }`; `ski_in_ski_out` skips Google; Google/key failures yield `api: []` with curated preserved.
- Rate limit `places` 60/min/IP on `/api/places/nearby`.
- Verification: `npm run lint`, `npm run build`, `npm run test:unit` (231 tests), `test:launch` (with `NEXT_PUBLIC_CONTACT_EMAIL` set locally).
- **Pending PO/Ops:** manual QA with real Google key + Supabase; GCP billing budget alert (AC 20).

### File List

- `content/content-model.md`
- `content/resorts/_TEMPLATE.md`
- `content/resorts/myoko-kogen.md`
- `web/.env.example`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/types.ts`
- `web/src/lib/content/schema.test.ts`
- `web/src/lib/places/categories.ts`
- `web/src/lib/places/categories.test.ts`
- `web/src/lib/places/types.ts`
- `web/src/lib/places/fieldMask.ts`
- `web/src/lib/places/fieldMask.test.ts`
- `web/src/lib/places/normalize.ts`
- `web/src/lib/places/normalize.test.ts`
- `web/src/lib/places/dedupe.ts`
- `web/src/lib/places/dedupe.test.ts`
- `web/src/lib/places/ttl.ts`
- `web/src/lib/places/ttl.test.ts`
- `web/src/lib/places/cache.ts`
- `web/src/lib/places/searchNearby.ts`
- `web/src/lib/places/nearby.ts`
- `web/src/lib/places/nearby.test.ts`
- `web/src/app/api/places/nearby/route.ts`
- `web/src/app/api/places/nearby/route.test.ts`
- `web/src/lib/rate-limit/limiter.ts`
- `web/src/lib/rate-limit/limiter.test.ts`
- `web/src/middleware.ts`
- `_powri/implementation-artifacts/sprint-status.yaml`
