baseline_commit: ef2d7ef

# Story 11.9: POI Detail UX — Desktop Split-View & List Simplification

Status: ready-for-dev

<!-- Follow-up to Story 11.3 — fixes a real z-index bug on desktop and replaces the full below-map list with a desktop rail + minimal mobile fallback. Depends on Mapbox migration (11.5) and reuses 11.4's display-logic split. -->

## Story

As a **visitor on desktop viewing the resort map**,
I want the map bigger and place details shown beside it instead of in a modal that gets visually broken by the map, and I want the long list below the map gone,
so that browsing nearby places on desktop feels intentional instead of buggy and cramped, and the page isn't dominated by a giant list.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) · UX [`ux-designs/ux-phase2/design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §MapSection, §Elevation — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration — provides the base `isolate` stacking-context fix this story's regression guard checks) and Story **11.4** (bug fixes — provides the shared `poiSheetDisplayFields` primary/secondary text split this story's rail panel reuses).

**Supersedes:** Story 11.3 AC 10 (single below-map list for all breakpoints) — this story is the amendment, 11.3's file is not edited retroactively.

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Desktop layout | **Split-view**: enlarge the map (both width **and** height, not just width) and add a **persistent detail rail** beside it. One component serves pin taps, list-row taps, and coordless-curated entries identically — no popup-anchor/clipping edge cases |
| Mobile | **Unchanged** — bottom sheet stays exactly as today |
| List removal | Full below-map list removed on **both** breakpoints. Desktop: list lives inside the rail (no extra page height). Mobile: keep a **short list containing only pinless curated entries** (usually 0-2 rows) — preserves 11.3 AC 9 discoverability without the "half the page" problem |

---

## Root cause being fixed (confirmed 2026-07-17)

Leaflet's (and now Mapbox's, defensively per 11.5 AC 5) internal DOM can carry z-index values well above this app's overlay scale (Leaflet: `leaflet-control: 800`, `leaflet-top/bottom: 1000`, vs. this app's max of `z-[70]` outside full-screen flows). `PoiSheet.tsx`'s desktop-centered variant (`md:items-center`, `z-[60]`/`z-[61]`) lands visually on top of the map's own screen position (since the visitor just tapped a pin *on* the map) — allowing the map's internal panes/controls to paint over parts of the modal. Mobile's bottom-sheet variant (`items-end`) avoids this because it's positioned at the bottom of the viewport, away from the map. Story 11.5 already isolates the map's stacking context (`isolate` CSS) as a defensive base fix — this story's AC 5 is a **regression guard**, not a re-fix.

---

## Acceptance Criteria

### A. Desktop split-view layout

1. **Given** viewport **≥768px** (existing `md:` breakpoint convention used throughout this codebase, e.g. `DesktopTopNav`)
   **When** the map section renders
   **Then** layout becomes two columns: an enlarged map (increase **both** height and width vs. the mobile-only 240px fixed height — exact desktop dimensions TBD by dev within the existing content max-width, but must be meaningfully larger than 240px tall, not just wider) on one side, and a persistent detail rail on the other

2. **Given** the desktop rail with **no place selected**
   **When** it renders
   **Then** it shows the POI list (curated first, then API, filtered to the active chip/category — same ordering as 11.3 AC 10) **inside the rail**, not below the map

3. **Given** a pin **or** a list row inside the rail is tapped (desktop)
   **When** a place is selected
   **Then** the **same rail** switches its content from the list to that place's detail view (badge / bold name / quoted highlight or vicinity / "Open in Google Maps" CTA — reusing 11.4's `poiSheetDisplayFields` shape) — **no** floating popup anchored to the pin, **no** full-viewport modal, on desktop

4. **Given** a coordless curated entry (no map pin, per 11.3 AC 9) is tapped from the rail's list (desktop)
   **When** selected
   **Then** the rail shows its detail exactly like a pinned place would — the rail is the **one** detail surface for every trigger type on desktop, with no special-casing

5. **Given** mobile (**<768px**)
   **When** a pin or list row is tapped
   **Then** the existing bottom-sheet `PoiSheet.tsx` behavior is **completely unchanged** by this story

6. **Given** the enlarged desktop map with its isolated stacking context (11.5 AC 5)
   **When** the rail renders beside it and a place is selected
   **Then** there is **no** z-index leakage — this is a regression check confirming 11.5's fix holds under the new, larger layout (manual QA: visually inspect map controls/markers never paint over the rail)

### B. List simplification

7. **Given** mobile (**<768px**)
   **When** places load
   **Then** the previous full unbounded list (which could render 60+ rows under "All") is **removed**; only a short list of places **without map pins** renders below the map (typically 0-2 rows — coordless curated entries only)

8. **Given** desktop (**≥768px**)
   **When** places load
   **Then** there is **no** separate below-map list element — the full list lives inside the rail (per AC 2)

9. **Given** the mobile pinless-only list has **zero** entries (the common case — most categories have coordinates)
   **When** rendered
   **Then** nothing renders (same empty-render convention as today's `PoiList` returning `null`)

### C. Shared display logic (no duplication)

10. **Given** `web/src/lib/map/poiSheetDisplay.ts` (from Story 11.4: `{ showRating, showPowriBadge, primaryText, secondaryText }`)
    **When** this story builds the desktop rail's detail panel
    **Then** it reuses this same function/shape — extract the badge/name/quoted-highlight-or-vicinity JSX into a shared presentational component (e.g. `PoiDetailContent.tsx`) used by **both** `PoiSheet.tsx` (mobile) and the new rail panel (desktop), rather than duplicating the markup

11. **Given** `googleMapsPlaceUrl(name, placeId)` (11.4) and the `external_link_clicked` tracking call
    **When** the rail's "Open in Google Maps" CTA is tapped
    **Then** it fires identically to the mobile sheet's CTA (same `link_type`, `destination_domain`, `resort_slug` properties) — reuse, don't reimplement

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [ ] **Unit (Vitest):** pinless-filter helper for the mobile mini-list (`placesWithoutCoordinates` — mirrors existing `placesWithCoordinates` in `mergePlaces.ts`)
- [ ] **Component / integration tests:** rail state transitions (list ↔ detail) at logic level; shared `PoiDetailContent` renders identically to what `PoiSheet` rendered pre-refactor (regression coverage for 11.4's fields)
- [ ] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [ ] **Manual QA (PO):** desktop — confirm map is visibly larger (both axes), rail shows list by default, tapping any pin/row swaps rail to detail with no visual glitches from the map underneath, coordless curated entries open correctly from the rail; mobile — confirm bottom sheet and mini pinless-list behavior unchanged from today

---

## Tasks / Subtasks

- [ ] `web/src/lib/map/mergePlaces.ts`: add `placesWithoutCoordinates()` helper (AC: 7, 9)
- [ ] Extract `PoiDetailContent.tsx` (badge/name/highlight-or-vicinity/CTA) from `PoiSheet.tsx`'s current JSX, reused by both surfaces (AC: 10, 11)
- [ ] `PoiSheet.tsx`: mobile-only now, renders `PoiDetailContent` inside the existing bottom-sheet chrome (AC: 5)
- [ ] New `web/src/components/map/ResortMapDetailRail.tsx` (desktop): list-or-detail rail, renders `PoiList`-equivalent by default and `PoiDetailContent` when a place is selected (AC: 2-4)
- [ ] `web/src/components/map/ResortMapSection.tsx`: `md:` split-view layout (bigger map + rail on desktop, unchanged stacked layout + bottom sheet + mini-list on mobile) (AC: 1, 5-8)
- [ ] `web/src/components/map/PoiList.tsx`: mobile variant filters to pinless-only (AC: 7); desktop's full list lives in the new rail component instead
- [ ] Manual z-index regression check against 11.5's `isolate` fix under the new larger layout (AC: 6)
- [ ] Run full verification commands

---

## Dev Notes

### Brownfield

| File | Current state | This story's change |
|------|----------------|----------------------|
| `web/src/components/map/PoiSheet.tsx` | Full detail markup (badge/name/highlight/CTA) + bottom-sheet/centered-modal chrome, used on both breakpoints | Detail markup extracted to `PoiDetailContent.tsx`; `PoiSheet.tsx` becomes mobile-only chrome wrapping it |
| `web/src/components/map/PoiList.tsx` | Renders every matched place, unbounded (`<ul>` of all curated+API rows) | Mobile: filtered to pinless-only. Desktop: superseded by the new rail's internal list (reuses the same row markup, ideally the same component with a `variant` prop rather than a fork) |
| `web/src/components/map/ResortMapSection.tsx:132-193` | Single-column layout: chips → map (240px) → attribution → fallback link → full list → conditional `PoiSheet` overlay | `md:` two-column split: chips → [map (enlarged) \| rail] on desktop; chips → map (240px) → attribution → fallback link → mini pinless-list → conditional `PoiSheet` overlay on mobile |
| `web/src/lib/map/mergePlaces.ts:88-95` | `placesWithCoordinates()` only | Add sibling `placesWithoutCoordinates()` |

### Why the rail, not an anchored in-map popup

The PO's original ask described an in-map popup tied to the pin. Evaluated and rejected in favor of the rail because: (1) `PoiSheet` already opens identically from pin taps **and** list-row taps (11.3 AC 10-11), and coordless curated entries (11.3 AC 9) have no pin to anchor a popup to — an anchored popup would need a fallback for those triggers anyway, landing back on a second interaction pattern; (2) anchored popups near map edges need their own clamping/reposition logic that the rail avoids entirely; (3) the rail directly uses the "make the map bigger" real estate the PO asked for, rather than fighting for space with a floating overlay.

### Architecture compliance

- [`ux-designs/ux-phase2/design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §MapSection/§Elevation described the *mobile* bottom-sheet spec only — this story's desktop rail is new ground, not documented in the Phase 2 UX spine yet. Note the gap; don't force-fit the sheet's mobile tokens onto the rail if they don't make sense (e.g. `{rounded.sheet}` bottom-sheet corner radius doesn't apply to an inline rail panel).
- Existing `md:` breakpoint convention (768px) used throughout (`DesktopTopNav`, `PoiSheet`'s own `md:items-center`) — reuse, don't invent a new breakpoint.

### Out of scope

- Mapbox migration itself — Story 11.5 (prerequisite, provides the isolation fix this story regression-checks)
- Pin shapes, chip model, radius/cap — Stories 11.6-11.8 (independent, can land before or after this one)
- Any UX spine doc update beyond noting the gap above (full desktop rail spec is a UX design task, not a dev-story task, if PO wants it formalized)

---

## Change Log

- 2026-07-17: Story created — PO follow-up request after confirming the desktop z-index bug and requesting map enlargement + list simplification.

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List
