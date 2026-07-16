baseline_commit: 72d7c04c9e98b1be9f96830bdaa4dc10b859b232

# Story 11.1: Resort Geo Fields & Curated Places Content Pipeline

Status: review

<!-- PO pilot: myoko-kogen — see § PO input -->

## Story

As a **developer**,
I want latitude/longitude and an optional curated-places list on every published resort,
So that the map centers correctly and can surface editorial picks without runtime geocoding.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Epic 9.1 (`places_cache` table exists — no code changes in 11.1)

**Blocks:** Stories **11.2** (Places proxy) and **11.3** (map UI)

**Reference only (not source of truth):** [`input_docs/story-curated-map-places.md`](../../../input_docs/story-curated-map-places.md)

---

## PO input

**Pilot resort:** `myoko-kogen` (`content/resorts/myoko-kogen.md`)

Apply this frontmatter block (other 19 resorts + `_TEMPLATE.md` get `curated_map_places: []`):

```yaml
curated_map_places:
  - place_id: "ChIJZeAuKX099l8R2sLmirw42ho"
    category: restaurant
    highlight: "A hidden gem serving unforgettable creative Japanese cuisine."
    tags: ["restaurant", "dinner", "dining experience"]
```

**PO data review (schema vs supplied YAML):**

| Field | Status |
|-------|--------|
| `place_id` | ✅ `ChIJ…` prefix valid |
| `category` | ✅ `restaurant` |
| `tags` | ✅ 3 tags; longest `"dining experience"` = 17 chars (≤ 20) |
| `highlight` | ✅ 61 chars (≤ 80) |

---

## Acceptance Criteria

### A. Geo fields — schema, content, validation

1. **Given** `content/content-model.md` §4.1a documents `latitude`, `longitude`, `map_default_zoom`  
   **When** an editor reads the doc  
   **Then** pin placement rule (official base / main gondola), WGS84 ranges, and default zoom 13 are explicit

2. **Given** `web/src/lib/content/schema.ts` and `types.ts`  
   **When** frontmatter is parsed  
   **Then** `latitude` (−90…90), `longitude` (−180…180), and `map_default_zoom` (8–18, default **13**) are typed on `Resort`

3. **Given** all **20** published `content/resorts/*.md` files  
   **When** `npm run test:launch` runs  
   **Then** every published resort has valid `latitude` and `longitude` (existing coords must be **re-verified** against official base/gondola — do not assume prior values are correct)

4. **Given** a **published** resort missing `latitude` or `longitude`  
   **When** `npm run build` loads resorts via `loadResorts.ts`  
   **Then** a **`console.warn`** is emitted (slug + field) — build does **not** fail

5. **Given** a **published** resort missing `latitude` or `longitude`  
   **When** `npm run test:launch` runs  
   **Then** the gate **hard-fails** (preserve existing `verify-launch-gates.ts` behaviour)

### B. `curated_map_places` — schema, content, validation

6. **Given** `content/content-model.md` §4.1a extended with `curated_map_places`  
   **When** an editor reads the doc  
   **Then** purpose, field shapes, category enum, `ChIJ…` place_id rule, and empty-array fallback are documented in the Map pin subsection (not a separate top-level §)

7. **Given** `web/src/lib/content/schema.ts` adds `CuratedMapPlaceSchema` and `types.ts` adds `CuratedMapCategory` + `CuratedMapPlace`  
   **When** frontmatter is parsed  
   **Then** `curated_map_places` defaults to `[]` and maps to `resort.curatedMapPlaces: CuratedMapPlace[]` (camelCase export per existing content conventions)

8. **Category enum** (required on each entry):  
   `ski_in_ski_out` · `accommodation` · `restaurant` · `supermarket` · `convenience` · `transport` · `onsen` · `attraction`

9. **Given** a resort with `curated_map_places: []`  
   **When** build and `test:launch` run  
   **Then** no curated-related warnings

10. **Given** a resort with an **invalid `category`** enum value  
    **When** Zod parses frontmatter  
    **Then** build **fails** (parse error — same as other schema violations)

11. **Given** a resort with curated entries where `place_id` does not start with `ChIJ`, or `highlight` > 80 chars, or `tags` length > 3, or any tag > 20 chars  
    **When** `npm run test:launch` runs  
    **Then** **non-blocking warnings** are emitted per resort/entry (do **not** fail the gate)

12. **Given** the built resort index  
    **When** `getResortBySlug(slug)` is called  
    **Then** `curatedMapPlaces` is always a typed array (empty or populated)

### C. Resort markdown rollout

13. **Given** `_TEMPLATE.md` and all 20 published resort files  
    **When** reviewed  
    **Then** each has `curated_map_places: []` (or populated pilot data — see PO input)  
    **And** commented YAML example lives in `_TEMPLATE.md` only

14. **Given** PO-supplied pilot resort YAML  
    **When** applied to the named `content/resorts/<slug>.md`  
    **Then** entries pass Zod and launch-gate soft checks

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `web/src/lib/content/schema.test.ts` (new) — `CuratedMapPlaceSchema` happy path; invalid category throws; soft-rule cases parse successfully (warnings tested in launch gate or separate validator unit if extracted)
- [x] **Content / resorts:** `npm run test:launch` — geo hard-fail + curated soft warnings
- [x] **Build:** `npm run build` — geo missing → `console.warn` only; no regression on existing resorts
- [x] **Lint:** `npm run lint`
- [x] **Analytics:** N/A (no events in 11.1)
- [x] **E2E:** N/A (no UI)
- [x] **Manual QA:** PO confirms pilot resort curated entries look correct in frontmatter; spot-check 2–3 other resorts for `curated_map_places: []`

---

## Tasks / Subtasks

- [x] **PO:** Pilot `myoko-kogen` + YAML supplied (AC: 14)
- [x] Extend `content/content-model.md` §4.1a — geo recap + `curated_map_places` subsection (AC: 1, 6)
- [x] Add `CuratedMapCategory`, `CuratedMapPlace`, `curatedMapPlaces` to `types.ts` (AC: 7, 12)
- [x] Add `CuratedMapPlaceSchema` + field to `resortFrontmatterSchema`; export if needed (AC: 7, 10)
- [x] Map `curated_map_places` → `curatedMapPlaces` in `loadResorts.ts` (AC: 12)
- [x] Add **build-time** `console.warn` for published resorts missing lat/lng in `loadResorts.ts` (AC: 4) — warn once per slug, not per request in dev
- [x] Add curated **soft validation** helper; call from `verify-launch-gates.ts` (AC: 11) — warnings only
- [x] Re-verify all 20 resort coordinates (official base/gondola); fix any drift (AC: 3)
- [x] Add `curated_map_places: []` to `_TEMPLATE.md` + all 20 resorts; apply PO pilot data (AC: 13, 14)
- [x] Add `schema.test.ts` Vitest coverage (AC: 10)
- [x] Run `npm run lint && npm run build && npm run test:launch && npm run test:unit`

---

## Dev Notes

### Current state (brownfield — do not redo blindly)

| Area | Status |
|------|--------|
| `latitude` / `longitude` / `map_default_zoom` in Zod + `Resort` type | ✅ Already present |
| `loadResorts.ts` mapping | ✅ Already maps geo fields |
| All 20 published `.md` files | ✅ Appear to have lat/lng — **must re-verify** per AC 3 |
| `verify-launch-gates.ts` geo check | ✅ Already **hard-fails** missing coords |
| `curated_map_places` | ❌ Not implemented anywhere |
| Build-time geo warn | ❌ Not implemented |

### Validation split (PO decision 2026-07-16)

| Issue | Channel | Fails? |
|-------|---------|--------|
| Published resort missing lat/lng | `loadResorts.ts` at build | Warn only |
| Published resort missing lat/lng | `verify-launch-gates.ts` | **Fail** |
| Invalid `category` | Zod parse at build | **Fail** |
| Bad `place_id` prefix / `highlight` / `tags` | `verify-launch-gates.ts` | Warn only |
| Empty `curated_map_places: []` | — | Silent OK |

Implement curated soft checks in **`verify-launch-gates.ts`** (not `loadResorts.ts`) to match PO choice. Extract a small pure validator in `web/src/lib/content/` if it keeps the gate script readable.

### `CuratedMapPlace` shape (Zod input = YAML keys)

```yaml
curated_map_places:
  - place_id: "ChIJxxxxxxxxxxxxxxx"   # required; must start with ChIJ
    category: restaurant               # required enum
    highlight: "Optional tip, max 80"  # optional; >80 chars → launch-gate warn (not Zod fail)
    tags: ["ski-in", "onsen"]          # optional; max 3 tags, each max 20 chars
```

**Zod note:** Do **not** put `.max(80)` on `highlight` in `CuratedMapPlaceSchema` — that would hard-fail build. Enforce 80-char limit only in `verify-launch-gates.ts` (warn).

**Export naming:** Follow existing camelCase transform pattern (`place_id` → keep as `placeId` **or** keep snake `place_id` on the type — match how `food_picks` → `FoodPick` fields are named (`name`, `cuisine`, not snake). Prefer **camelCase** on the exported `CuratedMapPlace` interface for consistency with `Resort` fields.

### `ski_in_ski_out`

Editorial-only category — no Google Places equivalent. Schema supports it in 11.1; API/UI behaviour is 11.2/11.3.

### Files to touch

| File | Change |
|------|--------|
| `content/content-model.md` | Extend §4.1a |
| `content/resorts/_TEMPLATE.md` | `curated_map_places: []` + comment block |
| `content/resorts/*.md` (×20) | `[]` + pilot population |
| `web/src/lib/content/types.ts` | Types + `Resort.curatedMapPlaces` |
| `web/src/lib/content/schema.ts` | `CuratedMapPlaceSchema` |
| `web/src/lib/content/loadResorts.ts` | Map field + geo warn |
| `web/scripts/verify-launch-gates.ts` | Curated soft warnings |
| `web/src/lib/content/schema.test.ts` | **New** unit tests |
| `web/src/lib/content/index.ts` | Re-export types if needed by 11.2 |

### Out of scope (11.2 / 11.3 / PO later)

- `/api/places/nearby` route
- `ResortMapSection` / Leaflet UI
- Populating curated data for all 20 resorts (only one pilot in 11.1)
- Google Places API key / `places_cache` TTL
- Analytics events `map_section_viewed` / `map_pin_tapped`
- "Powri pick" visual treatment (UX TBD — 11.3)

### Architecture compliance

- [`architecture/phase2/architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §3.1–3.2: editorial content stays in `content/resorts/*.md`; build-time Zod + SSG
- Addendum §K: geo fields; §L/F referenced by downstream stories only

### Previous epic intelligence

Epic 10 content-touch pattern: minimal schema changes, `test:launch` as content gate, camelCase transforms in Zod `.transform()`. No `curated_map_places` precedent — follow `foodPickSchema` / `accommodationPickSchema` patterns in `schema.ts`.

---

## Dev Agent Record

### Agent Model Used

Composer (dev-story — 2026-07-16)

### Debug Log References

- `test:launch` requires `NEXT_PUBLIC_CONTACT_EMAIL` in env (used `ci@example.com` locally, same as Playwright config).

### Completion Notes List

- Added `CuratedMapPlaceSchema` + `CuratedMapCategory` / `CuratedMapPlace` types; `curated_map_places` → `curatedMapPlaces` on `Resort`.
- Build-time geo warn (once per slug+field) in `loadResorts.ts`; curated soft checks in `validateCuratedMapPlaces.ts` + `verify-launch-gates.ts`.
- Extended `content/content-model.md` §4.1a; rolled out `curated_map_places` to all 20 resorts + `_TEMPLATE.md` (pilot on `myoko-kogen`).
- Re-verified coordinates for 20 resorts via official/resort GIS sources; updated 16 slugs with drift >~1 km (kept `kiroro`, `naeba`, `niseko-united`, `tsugaike-kogen` unchanged).
- Tests: `schema.test.ts`, `validateCuratedMapPlaces.test.ts`; `lint`, `build`, `test:launch`, `test:unit` all pass.

### File List

- `content/content-model.md`
- `content/resorts/_TEMPLATE.md`
- `content/resorts/*.md` (×20)
- `web/src/lib/content/types.ts`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/loadResorts.ts`
- `web/src/lib/content/validateCuratedMapPlaces.ts`
- `web/src/lib/content/index.ts`
- `web/src/lib/content/schema.test.ts`
- `web/src/lib/content/validateCuratedMapPlaces.test.ts`
- `web/scripts/verify-launch-gates.ts`
- `_powri/implementation-artifacts/sprint-status.yaml`
