baseline_commit: ef2d7ef

# Story 11.6: Category Pin Shapes

Status: ready-for-dev

<!-- Follow-up to Story 11.3 — distinguish API pins by category shape, not just color. Depends on Mapbox migration (11.5) for the marker rendering primitive. -->

## Story

As a **visitor browsing the resort map**,
I want each place category to have a visually distinct pin shape,
so that I can tell restaurants, transport, accommodation, etc. apart at a glance instead of every category looking like the same grey dot.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration) merged — pins render via `react-map-gl` `<Marker>` JSX children, not Leaflet `DivIcon` HTML strings.

**Blocks:** Story **11.7** needs an icon reserved for its new "Ski Resort" category (`MountainSnow`) — coordinate icon assignments so there's no collision (see Dev Notes).

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Icon source | `lucide-react` (already a project dependency — no new asset pipeline) |
| Curated ("Powri pick") pins | **Unchanged** — stay the black sparkle/star for every category (11.3 PO decision preserved). Only **API** pins get category shapes. |
| Resort-center pin | Unchanged (accent blue dot) |

---

## Acceptance Criteria

1. **Given** an API pin for each category
   **When** rendered
   **Then** it uses a distinct `lucide-react` icon inside the existing `{colors.map-pin-poi}` (#6E6A63) circular badge:
   - `restaurant` → `UtensilsCrossed`
   - `accommodation` → `Bed`
   - `supermarket` → `ShoppingCart`
   - `convenience` → `Store`
   - `onsen` → `Waves`
   - `attraction` → `Landmark` (**not** `Mountain`/`MountainSnow` — those are reserved for Story 11.7's new Ski Resort category; using them here would collide)
   - `transport` → `Bus`

2. **Given** a curated ("Powri pick") pin of any category
   **When** rendered
   **Then** it remains the black sparkle/star DivIcon-equivalent from 11.3 — category icons apply to **API pins only**

3. **Given** `web/src/components/map/mapIcons.ts`
   **When** rewritten
   **Then** it exports a category → icon lookup used by the API pin renderer, keeping the resort-center and curated-pin functions unchanged from what 11.5 ported

4. **Given** `web/src/lib/map/pinIcons.test.ts`
   **When** extended
   **Then** it asserts the category → icon mapping covers all 7 chip categories with no duplicates

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [ ] **Unit (Vitest):** category → icon mapping test (no duplicate icons across categories, all 7 covered)
- [ ] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [ ] **Manual QA (PO):** `myoko-kogen` — toggle each chip, confirm the API pins for that category show the right icon shape; curated sparkle pin still black/unchanged

---

## Tasks / Subtasks

- [ ] `web/src/components/map/mapIcons.ts`: category → `lucide-react` icon map for API pins (AC: 1, 3)
- [ ] Render the icon inside the existing grey circular badge markup (not a bare icon with no background) (AC: 1)
- [ ] `web/src/lib/map/pinIcons.test.ts`: mapping coverage/no-duplicates test (AC: 4)
- [ ] Run full verification commands

---

## Dev Notes

### Brownfield

| File | Current state (post-11.5) | This story's change |
|------|---------------------------|----------------------|
| `web/src/components/map/mapIcons.ts` | Ported from Leaflet: 3 functions (`resortCenterIcon`, `apiPoiIcon`, `curatedPoiIcon`) each returning one fixed marker visual | `apiPoiIcon(category)` takes a category param and swaps the inner icon glyph; `resortCenterIcon`/`curatedPoiIcon` untouched |

### Icon reservation table (coordinate with Story 11.7)

| Category | Icon | Reserved by |
|----------|------|-------------|
| restaurant | `UtensilsCrossed` | This story |
| accommodation | `Bed` | This story |
| supermarket | `ShoppingCart` | This story |
| convenience | `Store` | This story |
| onsen | `Waves` | This story |
| attraction | `Landmark` | This story |
| transport | `Bus` | This story |
| **ski_resort** | `MountainSnow` | **Story 11.7** — do not use for `attraction` |

### Out of scope

- Ski Resort category itself (chip, schema, content) — Story 11.7
- Pin size changes — none needed here (Story 11.7's Ski Resort pin is "slightly larger," specific to that one category)
- Any change to curated pin visuals — explicitly staying as the black sparkle

---

## Change Log

- 2026-07-17: Story created — PO follow-up request for per-category pin shapes.

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List
