baseline_commit: ef2d7ef

# Story 11.7: Chip "All" Interaction Model + Ski Resort Category

Status: ready-for-dev

<!-- Follow-up to Story 11.3 — replaces multi-select chips with All + single-select, adds a new curated-only "Ski Resort" category for valley resorts. Depends on Mapbox migration (11.5). -->

## Story

As a **visitor browsing the resort map**,
I want an "All" chip that shows everything and single-select category chips that narrow the view, plus a way to see the other ski resorts inside a multi-resort valley,
so that filtering feels predictable (one filter at a time, always a way back to "everything") and I can locate nearby linked resorts like Hakuba's 10 sub-areas.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration). Story **11.6** (pin shapes) recommended first — reserves `MountainSnow` for this story's Ski Resort icon (see the collision flag in AC 9 below).

**Supersedes:** Story 11.3 AC 5 (all 7 chips selected by default / multi-select union model) and AC 6 — this story is the amendment, 11.3's file is not edited retroactively.

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Chip model | **"All" + single-select categories** (radio-like) — tapping a category shows only it and turns off "All"/other chips; tapping "All" clears back to everything; **"All" cannot be deselected directly** |
| New chip | **"Ski Resort"** — inserted immediately after "All" (2nd position overall) |
| Ski Resort scope | **Pilot: Hakuba Valley + Myoko Kogen only** (both already have populated `linked_resorts`). Nozawa dropped — its `linked_resorts` is empty and wasn't the example intended. |
| Ski Resort data source | **Curated-only** (`callGoogle: false`, same pattern as `ski_in_ski_out`) — Google Places has no reliable "ski resort" place type to search |

---

## ⚠️ Flagged for PO — shape vs. sparkle conflict

Story 11.6 keeps curated ("Powri pick") pins as the **black sparkle for every category**, unchanged from 11.3. But Ski Resort pins are **curated-only** (this story), and the PO ask for this specific category was *"slightly larger pin with relevant shape (mountain peak etc.)"* — a shape, not a sparkle. Applying 11.6's general rule literally would make every Ski Resort pin a plain black sparkle, contradicting this story's own request.

**Default resolution used below (AC 9): Ski Resort is a deliberate exception to the "curated = sparkle" rule** — it renders as a slightly-larger `MountainSnow` glyph instead of the sparkle, specifically because the whole point of this category is shape-based recognition of multiple resorts clustered in one valley. **Confirm with PO before/during implementation** — if this is wrong, the fallback is: keep the sparkle for Ski Resort too, and rely on chip-filtering alone (not shape) to distinguish it.

---

## Acceptance Criteria

### A. Chip interaction model

1. **Given** first paint of the map section
   **When** chips render
   **Then** **9 chips** total, in this exact order: **All · Ski Resort · Restaurant · Accommodation · Supermarket · Convenience store · Onsen · Attraction · Transportation**
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
   **And** `apiCategoriesForChip(s)`/`apiCategoriesForChips` are adapted: `'all'` resolves to every Google-backed category (today's behavior when all 7 were selected); a single category resolves to just that category's API categories (unchanged per-chip mapping, e.g. Accommodation → `accommodation` + `ski_in_ski_out`)

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
    **Then** it uses the `MountainSnow` `lucide-react` icon (reserved in Story 11.6's icon table) at roughly **20-25% larger** than the standard curated sparkle size, in place of the sparkle — **pending PO confirmation** per the flag above

### D. Analytics

14. **Given** `map_pin_tapped`'s `place_category` property
    **When** a Ski Resort pin/list-row is tapped
    **Then** `place_category: 'ski_resort'` fires (new valid value — it does **not** collapse to another chip key, unlike `ski_in_ski_out` → `accommodation`)

15. **Given** `docs/analytics/tracking-plan.md` and `web/src/lib/analytics/phase2Events.ts`
    **When** updated
    **Then** `map_pin_tapped`'s notes list `ski_resort` alongside the existing chip-key enum, and `npm run test:analytics` passes

### E. i18n

16. **Given** `web/messages/en.json` `resortDetail.map.chips`
    **When** updated
    **Then** new keys `all` ("All") and `skiResort` ("Ski Resort") are added

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [ ] **Unit (Vitest):** `chips.test.ts` rewritten for the all/single-select model (AC: 7); `categories.test.ts` for `ski_resort` config; `schema.test.ts` for `ski_resort` enum value
- [ ] **Content:** `npm run test:launch` passes (works whether or not PO has backfilled `ski_resort` entries yet — empty is valid)
- [ ] **Analytics:** tracking plan + `phase2Events.ts` updated; `npm run test:analytics` passes
- [ ] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [ ] **Manual QA (PO):** `hakuba-valley` (once backfilled) — All shows everything, tapping Ski Resort shows only linked-resort pins (larger, mountain-shaped), tapping another category swaps cleanly, tapping the active chip again or otherwise clearing reverts to All

---

## Tasks / Subtasks

- [ ] `web/src/lib/map/chips.ts`: replace multi-select model with `'all' | MapChipKey` selection + `selectChip()` (AC: 1-6)
- [ ] `web/src/lib/map/chips.test.ts`: rewrite for new model (AC: 7)
- [ ] `web/src/components/map/CategoryChips.tsx`: render 9 chips in order, wire to new selection type (AC: 1)
- [ ] `web/src/components/map/ResortMapSection.tsx`: adapt state (`selectedChips: MapChipKey[]` → `selection: ChipSelection`), fetch orchestration for `'all'` vs single category (AC: 2, 6)
- [ ] `web/src/lib/content/types.ts` + `schema.ts`: add `ski_resort` to category enum (AC: 8)
- [ ] `web/src/lib/places/categories.ts`: `ski_resort` config, `callGoogle: false` (AC: 9)
- [ ] `web/src/lib/map/chips.ts`: `ski_resort` chip key, standalone (not merged into accommodation) (AC: 10)
- [ ] `content/content-model.md`: document `ski_resort` (AC: 11)
- [ ] Flag PO content task: backfill Hakuba + Myoko `curated_map_places` with `ski_resort` entries (AC: 12) — **manual, non-blocking**
- [ ] `web/src/components/map/mapIcons.ts`: `MountainSnow`, larger size, for `ski_resort` curated pins (AC: 13) — confirm sparkle-override decision with PO first
- [ ] `docs/analytics/tracking-plan.md` + `phase2Events.ts`: `ski_resort` in `place_category` notes (AC: 14, 15)
- [ ] `web/messages/en.json`: `chips.all`, `chips.skiResort` (AC: 16)
- [ ] Run full verification commands

---

## Dev Notes

### Brownfield

| File | Current state | This story's change |
|------|----------------|----------------------|
| `web/src/lib/map/chips.ts` | `MAP_CHIP_KEYS` (7, no "all"), `toggleChipSelection` = multi-select union with deselect-all→all-on | New `ChipSelection` type, `selectChip()`, `MAP_CHIP_KEYS` grows to 8 (adds `ski_resort`; "All" is a *selection state*, not a chip key/category) |
| `web/src/components/map/CategoryChips.tsx` | Renders `MAP_CHIP_KEYS.map(...)`, `aria-pressed` per chip, multi-select styling | Add an explicit "All" button + reorder; single-select `aria-pressed` semantics (radio-group pattern, similar to existing star-rating input pattern per `experience.md` accessibility floor) |
| `web/src/components/map/ResortMapSection.tsx:54-116` | `selectedChips: MapChipKey[]` state, `apiCategoriesForChips(selectedChips)` fetch, `mergePlacesForChips(payloads, selectedChips)` filter | Adapt to single active category (or all); fetch orchestration simplifies for the single-category case (no more up-to-7-parallel-category fetches except under "All") |
| `web/src/lib/content/types.ts:3-11`, `schema.ts:68-77` | 8-value enum ending `attraction` | Add `ski_resort` |
| `web/src/lib/places/categories.ts:8-20` | `PLACES_CATEGORY_MAP` keyed by 8 categories | Add `ski_resort` entry, `callGoogle: false` |

### Why "All" is a selection state, not a category

There's no Google/curated category called "all" — it's a UI-only concept meaning "union of every chip's results," matching what 11.3 originally did by default with all 7 chips on. Don't add `'all'` to `CuratedMapCategory`/`MAP_CHIP_KEYS` — model it as a distinct branch in the selection type instead, to avoid leaking a fake category into places/analytics code that expects real categories.

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

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List
