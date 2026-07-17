baseline_commit: ef2d7ef

# Story 11.4: Bug Fixes — Google Maps Link & Curated Place Name

Status: done

<!-- Found during manual QA of unmerged Story 11.3 (feature/11-3-resort-map-section-ui). Tracked as its own story per PO decision 2026-07-17 (kept independent of 11.3's record). -->

## Story

As a **visitor viewing a POI sheet on the resort map**,
I want "Open in Google Maps" to open the actual place, and Powri picks to show a real place name,
so that the map section is trustworthy and useful instead of showing a wrong location or an anonymous "Powri pick" card.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Blocks:** Nothing functionally, but 11.9 (POI detail UX overhaul) reuses this story's display-logic refactor — land this first.

---

## Bugs found (PO manual QA, 2026-07-17)

| # | Symptom | Root cause |
|---|---------|------------|
| 1 | "Open in Google Maps" opens a generic Bangkok view with no pin | `googleMapsPlaceUrl()` sends `query_place_id` **without** the required `query` parameter. Per [Google's Maps URLs spec](https://developers.google.com/maps/documentation/urls/get-started#search-action): *"For the search action, you must specify a `query`, but you may also specify a `query_place_id`."* Google's example format is `query=PLACE_NAME&query_place_id=PLACE_ID`. With no `query`, Maps has nothing to search for. |
| 2 | Powri pick sheet only shows `highlight` (or literal "Powri pick") — no place name | `CuratedMapPlace` has no `name` field at all (11.3 PO decision: "schema has no name — do not call Place Details to invent one"). Fixing bug 1 for curated pins requires the same field this bug asks for — **the two bugs share one root fix.** |

---

## Acceptance Criteria

### A. Curated schema — add required `name`

1. **Given** `content/content-model.md` §4.1a `curated_map_places` subsection
   **When** an editor reads the doc
   **Then** a new **required** `name` field (non-empty string) is documented alongside `place_id`

2. **Given** `curatedMapPlaceSchema` in `web/src/lib/content/schema.ts`
   **When** frontmatter is parsed
   **Then** `name: z.string().min(1)` is required (build fails, like `place_id`, if missing/empty) and maps to camelCase `name` on `CuratedMapPlace` (`web/src/lib/content/types.ts`)

3. **Given** the existing pilot curated entry in `content/resorts/myoko-kogen.md`
   **When** this story ships
   **Then** it is backfilled with `name: "Akakura Restaurant Shibata"` (per 11.2 Dev Agent Record completion notes — confirm exact display form with PO before merge; do not guess a different name)

4. **Given** `web/src/lib/content/schema.test.ts`
   **When** tests run
   **Then** a case asserts a curated entry missing `name` fails validation

### B. `MapPlaceItem` + merge — thread `name` through for curated

5. **Given** `web/src/lib/map/types.ts` `MapPlaceItem.name`
   **When** updated
   **Then** the `/** API only */` comment is removed — `name` is now populated for **both** sources

6. **Given** `web/src/lib/map/mergePlaces.ts` `curatedToItem()`
   **When** building a curated `MapPlaceItem`
   **Then** it includes `name: place.name`

### C. "Open in Google Maps" — fix the URL

7. **Given** `web/src/lib/map/googleMapsUrls.ts`
   **When** `googleMapsPlaceUrl` is called
   **Then** its signature becomes `googleMapsPlaceUrl(name: string, placeId: string): string` returning `https://www.google.com/maps/search/?api=1&query=<url-encoded name>&query_place_id=<placeId>` (both params always present, `query` first per Google's recommended format)

8. **Given** `PoiSheet.tsx`'s call site
   **When** building `mapsUrl`
   **Then** it passes the place's display name (from step D below) alongside `place.placeId` — for both API and curated places

9. **Given** `web/src/lib/map/googleMapsUrls.test.ts`
   **When** tests run
   **Then** the existing place-URL test is updated to assert both `query=` and `query_place_id=` are present and correctly encoded

### D. POI sheet display — badge, bold name, quoted highlight

10. **Given** `web/src/lib/map/poiSheetDisplay.ts`
    **When** `poiSheetDisplayFields(place)` is called
    **Then** it returns `{ showRating: false, showPowriBadge: boolean, primaryText: string | null, secondaryText: string | null }` where:
    - **API place:** `primaryText = place.name`, `secondaryText = place.vicinity ?? null`, `showPowriBadge = false` (unchanged behavior)
    - **Curated place:** `primaryText = place.name` (always present now — required), `secondaryText = place.highlight?.trim() || null` (raw, **not** pre-quoted — quoting is a display concern), `showPowriBadge = true`

11. **Given** `PoiSheet.tsx`
    **When** a curated place's sheet renders
    **Then** the order is: **"Powri pick" badge** (top) → **`primaryText` as bold heading** (`font-semibold`, existing `card-title` styles) → **`secondaryText` as a quoted subtitle** (e.g. `"{secondaryText}"`, `text-text-secondary`, smaller/lighter than the name) — only rendered when non-null (no empty quote line when highlight is empty)

12. **Given** `PoiSheet.tsx`
    **When** an API place's sheet renders
    **Then** unchanged: `primaryText` as heading, `secondaryText` (vicinity) as plain subtitle **without** quote marks

13. **Given** `web/src/lib/map/poiSheetDisplay.test.ts`
    **When** tests run
    **Then** cases cover: curated with highlight (both fields populated), curated with empty highlight (`secondaryText: null`, badge still true), API (unchanged, no badge)

### E. POI list — show real name

14. **Given** `PoiList.tsx`
    **When** a curated row renders
    **Then** primary text is the place's real `name` (not the literal "Powri pick" string), with `t('powriPick')` shown as the secondary/caption line — mirrors the API row's `name` + `vicinity` pattern for visual consistency

15. **Given** an API row
    **When** it renders
    **Then** unchanged (`name` primary, `vicinity` secondary)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `googleMapsUrls.test.ts` (query + query_place_id), `poiSheetDisplay.test.ts` (name/badge/quote fields), `schema.test.ts` (required `name`), `mergePlaces.test.ts` (curated `name` threading)
- [x] **Content:** `npm run test:launch` passes with `myoko-kogen.md` backfilled `place_name`
- [x] **Analytics:** N/A — no event/property changes
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA (PO):** `myoko-kogen` — tap curated restaurant pin/list row → sheet shows badge, bold name, quoted highlight → "Open in Google Maps" opens the correct restaurant (not Bangkok); tap an API pin → link opens the correct place too

---

## Tasks / Subtasks

- [x] `content/content-model.md` §4.1a: document required `place_name` (AC: 1) — PO decision: field is `place_name` (not `name`) to mirror `place_id`
- [x] `content/resorts/_TEMPLATE.md`: add `place_name` to the commented curated example
- [x] `web/src/lib/content/schema.ts` + `types.ts`: required `place_name` → `placeName` on curated schema/type (AC: 2)
- [x] `content/resorts/myoko-kogen.md`: PO backfilled pilot entry `place_name: "CHŌCHIN-ZA 提灯座"` (AC: 3)
- [x] `web/src/lib/content/schema.test.ts`: required-`place_name` test case (AC: 4)
- [x] `web/src/lib/map/types.ts`: `MapPlaceItem.name` shared, not API-only (AC: 5)
- [x] `web/src/lib/map/mergePlaces.ts`: thread `name` in `curatedToItem()` (AC: 6) + `mergePlaces.test.ts` update
- [x] `web/src/lib/map/googleMapsUrls.ts` + `.test.ts`: new `googleMapsPlaceUrl(name, placeId)` signature (AC: 7, 9)
- [x] `web/src/lib/map/poiSheetDisplay.ts` + `.test.ts`: new `{ primaryText, secondaryText }` shape (AC: 10, 13)
- [x] `web/src/components/map/PoiSheet.tsx`: badge → bold name → quoted highlight (curated) / plain vicinity (API); pass name into `googleMapsPlaceUrl` (AC: 8, 11, 12)
- [x] `web/src/components/map/PoiList.tsx`: curated row shows real name + "Powri pick" caption (AC: 14, 15)
- [x] Run full verification commands

---

## Dev Notes

### Brownfield — files this story modifies (read before editing)

| File | Current state | This story's change |
|------|---------------|----------------------|
| `web/src/lib/content/types.ts:13-20` | `CuratedMapPlace` has no `name` | Add required `name: string` |
| `web/src/lib/content/schema.ts:79-95` | `curatedMapPlaceSchema` has no `name` | Add `name: z.string().min(1)`, no `.optional()` |
| `web/src/lib/map/types.ts:8-23` | `name?` commented "API only" | Becomes shared, still optional in the type (curated always populates it at runtime, but keep the type permissive since `MapPlaceItem` is a shared shape) |
| `web/src/lib/map/mergePlaces.ts:13-25` | `curatedToItem()` omits `name` | Add `name: place.name` |
| `web/src/lib/map/googleMapsUrls.ts:1-4` | `googleMapsPlaceUrl(placeId)` — **buggy**, no `query` | `googleMapsPlaceUrl(name, placeId)` |
| `web/src/lib/map/poiSheetDisplay.ts` | Returns `primaryText` (highlight-or-null) + `showPowriBadge`, no name concept | Returns `primaryText` (name) + `secondaryText` (vicinity/highlight) |
| `web/src/components/map/PoiSheet.tsx:61-108` | Branches on `place.source` with duplicated JSX for name/highlight | Unify around `display.primaryText`/`secondaryText`; pass name to `googleMapsPlaceUrl` |
| `web/src/components/map/PoiList.tsx:22-27` | `primary = t('powriPick')` for curated, ignoring any name | `primary = place.name`, `secondary = t('powriPick')` |

### Why this is scoped separately from 11.3

11.3 is merged/mid-review with its own AC set already signed off. These are defects found in manual QA of that unmerged work — PO decision (2026-07-17) was to track them as an independent story rather than reopening 11.3's record, while still branching from `main` **after** 11.3 merges.

### Architecture compliance

- No architecture changes — this is a bug fix within 11.3's already-approved data flow (`{ curated, api, attribution }` envelope from 11.2 is untouched).
- Google Maps URL format: [Google Maps URLs — Search action](https://developers.google.com/maps/documentation/urls/get-started#search-action) — `query` required, `query_place_id` optional-but-recommended-together, `query` used only as fallback if the place ID can't be resolved.

### Out of scope

- Map library swap (Mapbox) — Story 11.5
- Pin shape/category icon changes — Story 11.6
- Chip model / Ski Resort category — Story 11.7
- Radius/pin-count tuning — Story 11.8
- Desktop split-view / list simplification — Story 11.9 (will reuse this story's `primaryText`/`secondaryText` display split)

---

## Change Log

- 2026-07-17: Story created from PO manual QA findings on unmerged Story 11.3.
- 2026-07-17: Dev started — PO chose `place_name` (frontmatter) / `placeName` (TS) over story draft `name`; myoko backfill left to PO.

---

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- PO clarification: use `place_name` in `curated_map_places`; PO will supply value in `myoko-kogen.md`.
- `npm run lint && npm run test:unit && npm run build` pass; build logs myoko parse warning until `place_name` added.
- `npm run test:launch` fails (19/20 resorts) until myoko `place_name` backfill.

- 2026-07-17: Dev complete — verification green; story moved to review.

### Completion Notes List

- Schema: required `place_name` → `CuratedMapPlace.placeName`; merged to `MapPlaceItem.name`.
- Google Maps URL: `googleMapsPlaceUrl(name, placeId)` sends both `query` and `query_place_id`.
- POI sheet: unified `primaryText`/`secondaryText`; curated highlight quoted; list shows real name + Powri pick caption.
- Pilot content: `myoko-kogen` `place_name: "CHŌCHIN-ZA 提灯座"` (PO-supplied).
- Verification: `lint`, `build`, `test:unit` (248 tests), `test:launch` (with contact email env) all pass.

### File List

- `content/content-model.md`
- `content/resorts/_TEMPLATE.md`
- `content/resorts/myoko-kogen.md`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/types.ts`
- `web/src/lib/content/schema.test.ts`
- `web/src/lib/content/validateCuratedMapPlaces.test.ts`
- `web/src/lib/map/types.ts`
- `web/src/lib/map/mergePlaces.ts`
- `web/src/lib/map/mergePlaces.test.ts`
- `web/src/lib/map/googleMapsUrls.ts`
- `web/src/lib/map/googleMapsUrls.test.ts`
- `web/src/lib/map/poiSheetDisplay.ts`
- `web/src/lib/map/poiSheetDisplay.test.ts`
- `web/src/lib/places/nearby.test.ts`
- `web/src/lib/places/dedupe.test.ts`
- `web/src/components/map/PoiSheet.tsx`
- `web/src/components/map/PoiList.tsx`
- `_powri/implementation-artifacts/sprint-status.yaml`
