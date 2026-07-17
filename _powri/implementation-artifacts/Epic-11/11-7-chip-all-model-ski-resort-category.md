baseline_commit: ef2d7ef

# Story 11.7: Chip "All" Interaction Model + Ski Resort Category

Status: done

<!-- Follow-up to Story 11.3 — replaces multi-select chips with All + single-select, adds a new curated-only "Ski Resort" category for valley resorts. Depends on Mapbox migration (11.5). -->

## Story

As a **visitor browsing the resort map**,
I want an "All" chip that shows everything and single-select category chips that narrow the view, plus a way to see the other ski resorts inside a multi-resort valley,
so that filtering feels predictable (one filter at a time, always a way back to "everything") and I can locate nearby linked resorts like Hakuba's 10 sub-areas.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration). Story **11.6** (pin shapes) recommended first — reserves `MountainSnow` for this story's Ski Resort icon (see the collision flag in AC 9 below).

**Supersedes:** Story 11.3 AC 5 (all 7 chips selected by default / multi-select union model), AC 6, and the Attraction/Transportation chip rows from 11.3's chip table — this story is the amendment, 11.3's file is not edited retroactively.

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Chip model | **"All" + single-select categories** (radio-like) — tapping a category shows only it and turns off "All"/other chips; tapping "All" clears back to everything; **"All" cannot be deselected directly** |
| New chip | **"Ski Resort"** — inserted immediately after "All" (2nd position overall) |
| Ski Resort scope | **Pilot: Hakuba Valley + Myoko Kogen only** (both already have populated `linked_resorts`). Nozawa dropped — its `linked_resorts` is empty and wasn't the example intended. |
| Ski Resort data source | **Curated-only** (`callGoogle: false`, same pattern as `ski_in_ski_out`) — Google Places has no reliable "ski resort" place type to search |

### PO decisions (2026-07-17 — follow-up)

| Topic | Decision |
|-------|----------|
| Ski Resort map pin icon | **Font Awesome Free Solid `fa-mountain`**, black fill, no background circle — replaces lucide `MountainSnow` / `CableCar` |
| Ski Resort pin size | ~22% larger than curated sparkle (27×27 container, 22px icon) — unchanged from prior PO confirm |
| Powri pick badge | **Not shown** for `ski_resort` entries in POI list or POI sheet — valley ski areas are geographic listings, not editorial picks |
| POI list secondary line | `ski_resort`: show optional `highlight` if present; otherwise no secondary line (never `"Powri pick"`) |

### PO decisions (2026-07-17 — remove Attraction & Transportation)

| Topic | Decision |
|-------|----------|
| Attraction & Transportation chips | **Removed from UI entirely** — no filter chips for these categories |
| Google Places for `attraction` / `transport` | **`callGoogle: false`** — never call Google Places for these categories |
| Rationale | Google results are mostly the ski resort itself; valley ski areas are covered by curated **`ski_resort`** entries |
| Schema enum | **Keep** `attraction` and `transport` in `CuratedMapCategory` / `curatedMapCategorySchema` for possible future curated-only entries — they are **not** chip keys and **not** Google-backed |

---

## ⚠️ Flagged for PO — shape vs. sparkle conflict

Story 11.6 keeps curated ("Powri pick") pins as the **black sparkle for every category**, unchanged from 11.3. But Ski Resort pins are **curated-only** (this story), and the PO ask for this specific category was *"slightly larger pin with relevant shape (mountain peak etc.)"* — a shape, not a sparkle. Applying 11.6's general rule literally would make every Ski Resort pin a plain black sparkle, contradicting this story's own request.

**Default resolution used below (AC 13): Ski Resort is a deliberate exception to the "curated = sparkle" rule** — it renders as a slightly-larger mountain glyph (Font Awesome solid `fa-mountain`, black fill) instead of the sparkle. PO confirmed 2026-07-17; follow-up AC 17 hides Powri pick badge for valley listings.

---

## Acceptance Criteria

### A. Chip interaction model

1. **Given** first paint of the map section
   **When** chips render
   **Then** **7 chips** total, in this exact order: **All · Ski Resort · Restaurant · Accommodation · Supermarket · Convenience store · Onsen**
   **And** **Attraction** and **Transportation** chips from Story 11.3 are **not** shown
   **And** "All" is selected by default; no category chip is highlighted

2. **Given** "All" is currently selected
   **When** the visitor taps any one category chip
   **Then** "All" deselects, that one category chip becomes the sole selected chip, and pins/list filter down to exactly that category (Accommodation still merges `accommodation` + `ski_in_ski_out` per 11.3 AC 6 — that merge behavior is unchanged, only the *selection* model changes from multi-select to single-select)

3. **Given** a category chip is currently the sole selected chip
   **When** the visitor taps a **different** category chip
   **Then** the previous one deselects and the new one becomes the sole selection (radio behavior — never two category chips active at once)

4. **Given** a category chip is currently the sole selected chip
   **When** the visitor taps that **same** chip again (or otherwise ends up with zero category chips selected)
   **Then** selection reverts to "All" (there is no valid "nothing selected" state)

5. **Given** "All" is currently selected
   **When** the visitor taps "All" again
   **Then** it is a no-op — "All" cannot be deselected directly (matches 11.3's original "deselect-all → reselect all" spirit, generalized to the new model)

6. **Given** `web/src/lib/map/chips.ts`
   **When** rewritten for this model
   **Then** `toggleChipSelection` (multi-select union) is replaced by a selection model with an explicit "all" state, e.g. `type ChipSelection = 'all' | MapChipKey`, plus `selectChip(current, tapped): ChipSelection` implementing AC 2-5
   **And** `apiCategoriesForChip(s)`/`apiCategoriesForChips` are adapted: `'all'` resolves to every **chip-backed** category (`MAP_CHIP_KEYS` — excludes `attraction`/`transport`); Google fetches only categories with `callGoogle: true` (restaurant, accommodation, supermarket, convenience, onsen); a single category resolves to just that category's API categories (unchanged per-chip mapping, e.g. Accommodation → `accommodation` + `ski_in_ski_out`)

7. **Given** existing chip tests (`chips.test.ts`)
   **When** updated
   **Then** they cover: default is `'all'`, tapping a category sets it as sole selection, tapping "All" while a category is active clears to `'all'`, tapping the active category again reverts to `'all'`, tapping "All" while already `'all'` is a no-op

### B. Ski Resort category (data + config)

8. **Given** `CuratedMapCategory` (`web/src/lib/content/types.ts`) and `curatedMapCategorySchema` (`web/src/lib/content/schema.ts`)
   **When** extended
   **Then** a new value `ski_resort` is added to the enum

9. **Given** `web/src/lib/places/categories.ts` `PLACES_CATEGORY_MAP`
   **When** extended
   **Then** `ski_resort: { includedTypes: [], callGoogle: false }` — never calls Google, exactly like `ski_in_ski_out`

10. **Given** `web/src/lib/map/chips.ts` `MAP_CHIP_KEYS` and `chipKeyForCategory`
    **When** extended
    **Then** `'ski_resort'` is added as its own chip key (it does **not** collapse into `accommodation` — it's a standalone chip, unlike `ski_in_ski_out`)

11. **Given** `content/content-model.md` §4.1a
    **When** updated
    **Then** `ski_resort` is documented in the category enum table, noting it is editorial-only like `ski_in_ski_out`

12. **Given** PO content authoring (manual, not a code gate — mirrors 11.2's "PO adds pilot coords" pattern)
    **When** this story's code ships
    **Then** the PO backfills `curated_map_places` entries with `category: ski_resort` for:
    - **`content/resorts/hakuba-valley.md`** — up to 10 entries (`Happo-one`, `Hakuba Goryu`, `Hakuba 47`, `Tsugaike Kogen`, `Iwatake`, `Norikura`, `Cortina`, `Sanosaka`, `Kashimayari`, `Jiigatake`)
    - **`content/resorts/myoko-kogen.md`** — up to 5 entries (`Akakura Onsen`, `Akakura Kanko`, `Ikenotaira`, `Suginohara`, `Seki Onsen`)
    each with required `name` (Story 11.4 dependency — `name` is required on all curated entries by the time this ships), `place_id`, `latitude`/`longitude`. An empty/partial list is non-blocking (same "curated is optional" posture as everywhere else in this epic) — do not treat incomplete PO content as a story failure.

### C. Ski Resort pin visual

13. **Given** a Ski Resort pin (curated-only, per the flagged conflict above)
    **When** rendered
    **Then** it uses **Font Awesome Free Solid `fa-mountain`** (`@fortawesome/react-fontawesome`), black fill, no background circle, at roughly **20–25% larger** than the standard curated sparkle size — in place of the sparkle (not lucide-react)

17. **Given** a `ski_resort` entry in the POI list or POI detail sheet
    **When** displayed
    **Then** it does **not** show the "Powri pick" badge or secondary `"Powri pick"` label — these are valley ski-area listings, not editorial hand picks
    **And** other curated categories (`restaurant`, `accommodation`, etc.) continue to show the Powri pick badge as before

### D. Analytics

14. **Given** `map_pin_tapped`'s `place_category` property
    **When** a Ski Resort pin/list-row is tapped
    **Then** `place_category: 'ski_resort'` fires (new valid value — it does **not** collapse to another chip key, unlike `ski_in_ski_out` → `accommodation`)

15. **Given** `docs/analytics/tracking-plan.md` and `web/src/lib/analytics/phase2Events.ts`
    **When** updated
    **Then** `map_pin_tapped`'s notes list `ski_resort` alongside the active chip-key enum (`ski_resort` \| `restaurant` \| `accommodation` \| `supermarket` \| `convenience` \| `onsen` — **not** `attraction` or `transport`), and `npm run test:analytics` passes

### E. i18n

16. **Given** `web/messages/en.json` `resortDetail.map.chips`
    **When** updated
    **Then** new keys `all` ("All") and `skiResort` ("Ski Resort") are added
    **And** `attraction` and `transport` chip label keys are **removed** (chips no longer exist)

### F. Remove Attraction & Transportation (Google + chips)

18. **Given** `web/src/lib/map/chips.ts` `MAP_CHIP_KEYS`
    **When** updated for this follow-up
    **Then** `attraction` and `transport` are **removed** — `MAP_CHIP_KEYS` has **6** category keys: `ski_resort`, `restaurant`, `accommodation`, `supermarket`, `convenience`, `onsen`

19. **Given** `web/src/lib/places/categories.ts` `PLACES_CATEGORY_MAP`
    **When** updated
    **Then** `attraction: { includedTypes: [], callGoogle: false }` and `transport: { includedTypes: [], callGoogle: false }` — same curated-only posture as `ski_in_ski_out` / `ski_resort`; **no** Google Nearby Search for these categories

20. **Given** `CuratedMapCategory` and `curatedMapCategorySchema`
    **When** inspected after this story ships
    **Then** `attraction` and `transport` **remain** valid enum values (future curated-only entries allowed) but are **excluded** from `MAP_CHIP_KEYS`, chip UI, and Google fetch orchestration

21. **Given** `web/src/lib/map/categoryVisuals.ts` and related pin/chip styling
    **When** updated
    **Then** `attraction` / `transport` chip visuals are removed or unreachable from the map UI (no dead chip labels in `CategoryChips`)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `chips.test.ts` rewritten for the all/single-select model (AC: 7); `categories.test.ts` for `ski_resort` config; `schema.test.ts` for `ski_resort` enum value
- [x] **Unit (Vitest):** `chips.test.ts` — `MAP_CHIP_KEYS` excludes `attraction`/`transport`; `categories.test.ts` — both have `callGoogle: false` (AC: 18–19)
- [x] **Content:** `npm run test:launch` passes (works whether or not PO has backfilled `ski_resort` entries yet — empty is valid)
- [x] **Analytics:** tracking plan + `phase2Events.ts` updated; `npm run test:analytics` passes
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA (PO):** `hakuba-valley` (once backfilled) — All shows everything, tapping Ski Resort shows only linked-resort pins (larger, mountain-shaped), tapping another category swaps cleanly, tapping the active chip again or otherwise clearing reverts to All; **no Attraction or Transportation chips**; Network tab shows **no** `/api/places/nearby?category=attraction` or `category=transport` calls

---

## Tasks / Subtasks

- [x] `web/src/lib/map/chips.ts`: replace multi-select model with `'all' | MapChipKey` selection + `selectChip()` (AC: 1-6)
- [x] `web/src/lib/map/chips.test.ts`: rewrite for new model (AC: 7)
- [x] `web/src/components/map/CategoryChips.tsx`: render chips in order, wire to new selection type (AC: 1) — **amend:** 7 chips (All + 6 categories), no Attraction/Transportation (AC: 1, 18, 21)
- [x] `web/src/components/map/ResortMapSection.tsx`: adapt state (`selectedChips: MapChipKey[]` → `selection: ChipSelection`), fetch orchestration for `'all'` vs single category (AC: 2, 6)
- [x] `web/src/lib/content/types.ts` + `schema.ts`: add `ski_resort` to category enum (AC: 8)
- [x] `web/src/lib/places/categories.ts`: `ski_resort` config, `callGoogle: false` (AC: 9)
- [x] `web/src/lib/map/chips.ts`: `ski_resort` chip key, standalone (not merged into accommodation) (AC: 10)
- [x] `content/content-model.md`: document `ski_resort` (AC: 11)
- [x] Flag PO content task: backfill Hakuba + Myoko `curated_map_places` with `ski_resort` entries (AC: 12) — **manual, non-blocking**
- [x] `web/src/components/map/mapIcons.tsx`: Font Awesome solid `fa-mountain`, black fill, for `ski_resort` map pins (AC: 13)
- [x] `web/src/lib/map/curatedDisplay.ts` + `PoiList.tsx` + `poiSheetDisplay.ts`: hide Powri pick badge for `ski_resort` (AC: 17)
- [x] `docs/analytics/tracking-plan.md` + `phase2Events.ts`: `ski_resort` in `place_category` notes (AC: 14, 15)
- [x] `web/messages/en.json`: `chips.all`, `chips.skiResort` (AC: 16)
- [x] `web/src/lib/map/chips.ts`: remove `attraction` / `transport` from `MAP_CHIP_KEYS` (AC: 18)
- [x] `web/src/lib/places/categories.ts`: `callGoogle: false` for `attraction` and `transport` (AC: 19)
- [x] `web/src/lib/map/categoryVisuals.ts`: remove unused Attraction/Transportation chip visuals (AC: 21)
- [x] `web/messages/en.json`: remove `chips.attraction`, `chips.transport` (AC: 16)
- [x] `docs/analytics/tracking-plan.md` + `phase2Events.ts`: drop `attraction` / `transport` from `place_category` chip-key enum (AC: 15)
- [x] Run full verification commands (incl. amended chip count)

---

## Dev Notes

### Brownfield

| File | Current state | This story's change |
|------|----------------|----------------------|
| `web/src/lib/map/chips.ts` | `MAP_CHIP_KEYS` (7, no "all"), `toggleChipSelection` = multi-select union with deselect-all→all-on | New `ChipSelection` type, `selectChip()`, `MAP_CHIP_KEYS` = **6** category keys (`ski_resort` + 5 Google-backed; **drops** `attraction`/`transport`; "All" is a *selection state*, not a chip key/category) |
| `web/src/components/map/CategoryChips.tsx` | Renders `MAP_CHIP_KEYS.map(...)`, `aria-pressed` per chip, multi-select styling | Add an explicit "All" button + reorder; single-select `aria-pressed` semantics (radio-group pattern, similar to existing star-rating input pattern per `experience.md` accessibility floor) |
| `web/src/components/map/ResortMapSection.tsx:54-116` | `selectedChips: MapChipKey[]` state, `apiCategoriesForChips(selectedChips)` fetch, `mergePlacesForChips(payloads, selectedChips)` filter | Adapt to single active category (or all); fetch orchestration simplifies for the single-category case (no more up-to-7-parallel-category fetches except under "All") |
| `web/src/lib/content/types.ts:3-11`, `schema.ts:68-77` | 8-value enum ending `attraction` | Add `ski_resort` |
| `web/src/lib/places/categories.ts:8-20` | `PLACES_CATEGORY_MAP` keyed by 8 categories | Add `ski_resort` entry, `callGoogle: false`; set `attraction`/`transport` to `callGoogle: false` (schema enum retained, no chip, no Google fetch) |

### Why "All" is a selection state, not a category

There's no Google/curated category called "all" — it's a UI-only concept meaning "union of every chip's results." Don't add `'all'` to `CuratedMapCategory`/`MAP_CHIP_KEYS` — model it as a distinct branch in the selection type instead, to avoid leaking a fake category into places/analytics code that expects real categories.

### Why Attraction & Transportation are removed (not just Google-disabled)

Google Nearby Search for `tourist_attraction` and station types overwhelmingly returns the ski resort or lift infrastructure — duplicate noise now that **`ski_resort`** curated entries cover valley sub-areas. Removing the chips avoids an empty or misleading filter; keeping schema enum values allows future editorial `curated_map_places` with those categories if needed (they would appear under "All" only, not as dedicated chips).

### Content authoring note (Ski Resort pilot)

This mirrors 11.2's pattern where PO manually added `myoko-kogen`'s first curated pin post-code-ship. Do not block this story's Definition of Done on the PO having actually filled in `ski_resort` entries — the code must work correctly with **zero** entries (renders an empty-but-functional Ski Resort chip, same as any category with no curated/API results today) and equally correctly once PO backfills them later.

### Out of scope

- Google-Places-backed fallback for Ski Resort — explicitly rejected (PO confirmed curated-only)
- Nozawa Onsen — dropped from this story's scope; if PO wants it later, `linked_resorts` needs populating first (separate content task)
- Radius/pin-count tuning — Story 11.8
- Desktop split-view — Story 11.9

---

## Change Log

- 2026-07-17: Story created — PO follow-up request for All+single-select chips and a Ski Resort category for valley resorts.
- 2026-07-17: Follow-up — FA solid `mountain` pin; hide Powri pick badge for `ski_resort` list/sheet (AC 17).
- 2026-07-17: Implemented AC 18–21 — removed Attraction/Transportation chips; disabled Google for schema-only categories.

---

## Dev Agent Record

### Agent Model Used

Composer (AC 18–21 follow-up)

### Debug Log References

- PO confirmed AC 13: sparkle override; follow-up: FA solid `fa-mountain` black fill, no circle, ~22% larger than sparkle; chip color `text-primary`.
- AC 17: `ski_resort` hides Powri pick badge in list + sheet; list secondary uses highlight only when present.
- AC 18–21: removed Attraction/Transportation chips; `callGoogle: false` for schema-only categories; `googleBackedCategoriesForChips` + fetch fallback now use `getPlacesCategoryConfig().callGoogle`.

### Completion Notes List

- Replaced multi-select `toggleChipSelection` with `ChipSelection = 'all' | MapChipKey` and `selectChip()`.
- Chip UI: All (selection state) + **6** category chips (`ski_resort` first after All) — no Attraction/Transportation (AC 18–21).
- `ski_resort` is curated-only (`callGoogle: false`); standalone chip key for analytics (not merged into accommodation).
- `attraction` / `transport`: schema enum retained; removed from `MAP_CHIP_KEYS`, chip labels, analytics enum, and Google Places (`callGoogle: false`).
- `SkiResortCuratedPin`: Font Awesome solid `fa-mountain`, 27×27 container, 22px black fill.
- `isPowriPick()` excludes `ski_resort` from Powri pick badge in `PoiList` and `PoiSheet`.
- Added `@fortawesome/fontawesome-svg-core`, `@fortawesome/free-solid-svg-icons`, `@fortawesome/react-fontawesome`.
- Verification: lint, build, test:unit (261), test:analytics pass.

### File List

- `web/package.json`
- `web/package-lock.json`
- `web/src/lib/map/chips.ts`
- `web/src/lib/map/chips.test.ts`
- `web/src/lib/map/categoryVisuals.ts`
- `web/src/lib/map/curatedDisplay.ts`
- `web/src/lib/map/curatedDisplay.test.ts`
- `web/src/lib/map/mergePlaces.ts`
- `web/src/lib/map/mergePlaces.test.ts`
- `web/src/lib/map/fetchNearby.ts`
- `web/src/lib/map/pinIcons.test.ts`
- `web/src/lib/map/poiSheetDisplay.ts`
- `web/src/lib/map/poiSheetDisplay.test.ts`
- `web/src/lib/content/types.ts`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/schema.test.ts`
- `web/src/lib/places/categories.ts`
- `web/src/lib/places/categories.test.ts`
- `web/src/lib/analytics/phase2Events.ts`
- `web/src/components/map/CategoryChips.tsx`
- `web/src/components/map/ResortMapSection.tsx`
- `web/src/components/map/ResortMapboxMap.tsx`
- `web/src/components/map/mapIcons.tsx`
- `web/src/components/map/PoiList.tsx`
- `web/src/app/api/places/nearby/route.ts`
- `web/src/app/api/places/nearby/route.test.ts`
- `web/messages/en.json`
- `content/content-model.md`
- `docs/analytics/tracking-plan.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
