baseline_commit: ef2d7ef

# Story 11.9: POI Detail UX — Desktop Split-View & List Simplification

Status: done

<!-- Follow-up to Story 11.3 — fixes a real z-index bug on desktop and replaces the full below-map list with a desktop rail + chip-gated list. Depends on Mapbox migration (11.5) and reuses 11.4's display-logic split. -->

## Story

As a **visitor on desktop viewing the resort map**,
I want the map bigger and place details shown beside it instead of in a modal that gets visually broken by the map, and I want the long list below the map gone when browsing all categories,
so that browsing nearby places on desktop feels intentional instead of buggy and cramped, and the page isn't dominated by a giant list.

**Source shard(s):** [`epics/phase2/shards/epic-11-resort-map.md`](../../planning-artifacts/epics/phase2/shards/epic-11-resort-map.md) · [`prds/phase2/shards/05-features-resort-map.md`](../../planning-artifacts/prds/phase2/shards/05-features-resort-map.md) · UX [`ux-designs/ux-phase2/design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §MapSection, §Elevation — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **11.5** (Mapbox migration — provides the base `isolate` stacking-context fix this story's regression guard checks) and Story **11.4** (bug fixes — provides the shared `poiSheetDisplayFields` primary/secondary text split this story's rail panel reuses).

**Supersedes:** Story 11.3 AC 10 (single below-map list for all breakpoints) — this story is the amendment, 11.3's file is not edited retroactively.

---

## PO decisions (2026-07-17)

| Topic | Decision |
|-------|----------|
| Desktop layout | **Split-view on demand**: map is **full-width** when **All** is selected; selecting a category chip or tapping a pin **animates** the map narrower (left-anchored, same 400px height) while a detail rail **slides in from the right** (320px). Returning to **All** (or clearing the last open detail when on All) reverses the animation — rail slides out, map expands to full container width |
| Mobile | **Unchanged** — bottom sheet stays exactly as today |
| List visibility | **Chip-gated on both breakpoints**: hide the list when **All** is selected; show a filtered list (all matching places for the active chip, pinned + pinless-with-coords) when a category chip is selected. Desktop list lives in the rail; mobile list stays below the map |
| Coordless curated | **Not shown in lists** — coordless curated entries are omitted from list surfaces entirely (PO decision 2026-07-17 clarification) |
| Desktop rail on All | **No rail column** when All is selected and no pin/detail is open — map uses the full content width (not a blank rail panel) |

---

## Root cause being fixed (confirmed 2026-07-17)

Leaflet's (and now Mapbox's, defensively per 11.5 AC 5) internal DOM can carry z-index values well above this app's overlay scale (Leaflet: `leaflet-control: 800`, `leaflet-top/bottom: 1000`, vs. this app's max of `z-[70]` outside full-screen flows). `PoiSheet.tsx`'s desktop-centered variant (`md:items-center`, `z-[60]`/`z-[61]`) lands visually on top of the map's own screen position (since the visitor just tapped a pin *on* the map) — allowing the map's internal panes/controls to paint over parts of the modal. Mobile's bottom-sheet variant (`items-end`) avoids this because it's positioned at the bottom of the viewport, away from the map. Story 11.5 already isolates the map's stacking context (`isolate` CSS) as a defensive base fix — this story's AC 5 is a **regression guard**, not a re-fix.

---

## Acceptance Criteria

### A. Desktop split-view layout

1. **Given** viewport **≥768px** (existing `md:` breakpoint convention used throughout this codebase, e.g. `DesktopTopNav`)
   **When** the map section renders with **All** selected and no place open
   **Then** the map is **full container width** at **400px** height — **no** rail column is visible

1b. **Given** desktop with **All** selected
    **When** the visitor taps a map pin
    **Then** the map **animates narrower** (left-anchored, same height) and the detail rail **slides in from the right** showing that place's detail — **no** full-viewport modal

1c. **Given** desktop with the rail open (category chip or open pin/detail)
    **When** the visitor selects **All** again (or closes the last open detail while on All)
    **Then** the rail **slides out to the right** and the map **animates back to full container width** (300ms ease-in-out transition)

2. **Given** the desktop rail with **no place selected** and a **category chip** active (not All)
   **When** it renders
   **Then** it shows the POI list (curated first, then API, filtered to the active chip — same ordering as 11.3 AC 10; coordless curated entries excluded) **inside the rail**, not below the map

3. **Given** a category chip is selected on desktop (not All)
   **When** the rail opens
   **Then** the map and rail animate in together per AC 1b/1c (grid column transition + rail translate)

4. **Given** a pin **or** a list row inside the rail is tapped (desktop)
   **When** a place is selected
   **Then** the **same rail** switches its content from the list to that place's detail view (badge / bold name / quoted highlight or vicinity / "Open in Google Maps" CTA — reusing 11.4's `poiSheetDisplayFields` shape) — **no** floating popup anchored to the pin, **no** full-viewport modal, on desktop

5. **Given** mobile (**<768px**)
   **When** a pin or list row is tapped
   **Then** the existing bottom-sheet `PoiSheet.tsx` behavior is **completely unchanged** by this story

6. **Given** the enlarged desktop map with its isolated stacking context (11.5 AC 5)
   **When** the rail renders beside it and a place is selected
   **Then** there is **no** z-index leakage — this is a regression check confirming 11.5's fix holds under the new, larger layout (manual QA: visually inspect map controls/markers never paint over the rail)

### B. List simplification

7. **Given** mobile (**<768px**) with **All** chip selected
   **When** places load
   **Then** no list renders below the map (replacing the previous full unbounded list under All)

8. **Given** mobile (**<768px**) with a **category chip** selected (not All)
   **When** places load
   **Then** a filtered list of all matching places for that chip renders below the map (curated first, then API; coordless curated entries excluded)

9. **Given** desktop (**≥768px**)
   **When** places load
   **Then** there is **no** separate below-map list element — the chip-gated list lives inside the rail (per AC 2-3)

10. **Given** a chip-gated list (mobile or rail) has **zero** eligible entries after filtering
    **When** rendered
    **Then** nothing renders (same empty-render convention as today's `PoiList` returning `null`)

### C. Shared display logic (no duplication)

11. **Given** `web/src/lib/map/poiSheetDisplay.ts` (from Story 11.4: `{ showRating, showPowriBadge, primaryText, secondaryText }`)
    **When** this story builds the desktop rail's detail panel
    **Then** it reuses this same function/shape — extract the badge/name/quoted-highlight-or-vicinity JSX into a shared presentational component (e.g. `PoiDetailContent.tsx`) used by **both** `PoiSheet.tsx` (mobile) and the new rail panel (desktop), rather than duplicating the markup

12. **Given** `googleMapsPlaceUrl(name, placeId)` (11.4) and the `external_link_clicked` tracking call
    **When** the rail's "Open in Google Maps" CTA is tapped
    **Then** it fires identically to the mobile sheet's CTA (same `link_type`, `destination_domain`, `resort_slug` properties) — reuse, don't reimplement

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `placesForList()` helper — returns empty on All; returns coordinate-backed places only when a chip is active
- [x] **Component / integration tests:** rail state transitions (list ↔ detail) at logic level; shared `PoiDetailContent` renders identically to what `PoiSheet` rendered pre-refactor (regression coverage for 11.4's fields)
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA (PO):** desktop — confirm map is visibly larger (both axes), rail is blank on All, rail shows filtered list when a chip is selected, tapping any pin/row swaps rail to detail with no visual glitches from the map underneath; mobile — confirm bottom sheet unchanged, list hidden on All and filtered on chip select

---

## Tasks / Subtasks

- [x] `web/src/lib/map/mergePlaces.ts`: add `placesForList()` helper (AC: 7-10)
- [x] Extract `PoiDetailContent.tsx` (badge/name/highlight-or-vicinity/CTA) from `PoiSheet.tsx`'s current JSX, reused by both surfaces (AC: 11, 12)
- [x] `PoiSheet.tsx`: mobile-only now, renders `PoiDetailContent` inside the existing bottom-sheet chrome (AC: 5)
- [x] New `web/src/components/map/ResortMapDetailRail.tsx` (desktop): blank / list / detail states, renders `PoiList` when chip active and `PoiDetailContent` when a place is selected (AC: 2-4)
- [x] `web/src/components/map/ResortMapSection.tsx`: `md:` split-view layout (bigger map + rail on desktop, stacked layout + chip-gated below-map list + bottom sheet on mobile) (AC: 1, 5-9)
- [x] `web/src/components/map/PoiList.tsx`: optional rail styling variant; list callers pass `placesForList()` output (AC: 7-10)
- [x] Manual z-index regression check against 11.5's `isolate` fix under the new larger layout (AC: 6)
- [x] Run full verification commands

---

## Dev Notes

### Brownfield

| File | Current state | This story's change |
|------|----------------|----------------------|
| `web/src/components/map/PoiSheet.tsx` | Full detail markup (badge/name/highlight/CTA) + bottom-sheet/centered-modal chrome, used on both breakpoints | Detail markup extracted to `PoiDetailContent.tsx`; `PoiSheet.tsx` becomes mobile-only bottom-sheet chrome wrapping it |
| `web/src/components/map/PoiList.tsx` | Renders every matched place, unbounded (`<ul>` of all curated+API rows) | Hidden when All selected; chip-gated filtered list (coordless excluded) on mobile below map and in desktop rail |
| `web/src/components/map/ResortMapSection.tsx:132-193` | Single-column layout: chips → map (240px) → attribution → fallback link → full list → conditional `PoiSheet` overlay | `md:` two-column split: chips → [map (enlarged) \| rail] on desktop; chips → map (240px) → attribution → fallback link → chip-gated list → conditional mobile `PoiSheet` overlay |
| `web/src/lib/map/mergePlaces.ts:88-95` | `placesWithCoordinates()` only | Add `placesForList(places, selection)` — empty on All, else `placesWithCoordinates(places)` |

### Why the rail, not an anchored in-map popup

The PO's original ask described an in-map popup tied to the pin. Evaluated and rejected in favor of the rail because: (1) `PoiSheet` already opens identically from pin taps **and** list-row taps (11.3 AC 10-11); (2) anchored popups near map edges need their own clamping/reposition logic that the rail avoids entirely; (3) the rail directly uses the "make the map bigger" real estate the PO asked for, rather than fighting for space with a floating overlay.

### Architecture compliance

- [`ux-designs/ux-phase2/design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §MapSection/§Elevation described the *mobile* bottom-sheet spec only — this story's desktop rail is new ground, not documented in the Phase 2 UX spine yet. Note the gap; don't force-fit the sheet's mobile tokens onto the rail if they don't make sense (e.g. `{rounded.sheet}` bottom-sheet corner radius doesn't apply to an inline rail panel).
- Existing `md:` breakpoint convention (768px) used throughout (`DesktopTopNav`, `PoiSheet`'s own `md:items-center`) — reuse, don't invent a new breakpoint.

### Out of scope

- Mapbox migration itself — Story 11.5 (prerequisite, provides the isolation fix this story regression-checks)
- Pin shapes, chip model, radius/cap — Stories 11.6-11.8 (independent, can land before or after this one)
- Coordless curated discoverability — explicitly dropped per PO clarification 2026-07-17
- Any UX spine doc update beyond noting the gap above (full desktop rail spec is a UX design task, not a dev-story task, if PO wants it formalized)

---

## Change Log

- 2026-07-17: Story created — PO follow-up request after confirming the desktop z-index bug and requesting map enlargement + list simplification.
- 2026-07-17: PO clarification — chip-gated list on both breakpoints (hide on All, show filtered list on chip); blank desktop rail on All; coordless curated entries omitted from lists entirely.
- 2026-07-17: PO follow-up — collapsible animated rail on desktop (full-width map on All; rail slides in on chip/pin, slides out on return to All).
- 2026-07-17: CSS cascade fix — removed conflicting `grid-cols-1` utility; `.map-rail-layout.grid[data-rail-open]` owns column sizing on desktop only.

---

## Lessons learned (for Epic 11 retrospective)

### Cross-breakpoint layout regression (mobile fix broke desktop)

**What happened:** Fixing mobile map shrink (inline `gridTemplateColumns` driven by `railOpen` at all breakpoints) replaced inline styles with `.map-rail-layout[data-rail-open]` CSS scoped to `@media (min-width: 768px)`. Desktop map stop shrinking because **`grid-cols-1` Tailwind utility** on the same grid container overrode the custom column rules (inline styles had previously won).

**Root cause:** One shared layout primitive served two masters — mobile (always single column) and desktop (animated 1fr ↔ 1fr+320px). A fix targeting mobile side-effects was not re-verified against desktop AC.

**Resolution:** Remove `grid-cols-1`; let `.map-rail-layout.grid[data-rail-open='…']` own `grid-template-columns` (base = single column for mobile; `@media md+` = animated two-column).

### Practices for future stories (especially AI dev agents)

1. **Split AC by breakpoint** when mobile and desktop behavior diverge — avoid a single vague “responsive layout” AC.
2. **Verification matrix in DoD** — table of actions × expected desktop vs mobile outcome; re-check both after any follow-up fix.
3. **Don't mix Tailwind layout utilities with custom column CSS** on the same element — pick one owner for `grid-template-columns` and document it in a one-line comment at the layout site.
4. **Scope breakpoint-specific behavior in CSS** (`@media`) or separate wrappers — don't use one inline style / shared utility for both when only one breakpoint should animate.
5. **Follow-up fixes must re-read all AC**, not only the latest user message — state explicitly which sibling AC are preserved and how.
6. **Checkpoint before review** — PO or `bmad-checkpoint-preview` pass with “mobile fix — confirm desktop Y still holds.”

**Not a fundamental conflict:** Desktop rail animation and mobile static map **can coexist** in one DOM tree when column sizing is breakpoint-scoped correctly.

---

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

### Completion Notes List

- Added `placesForList()` (empty on All; coordinate-backed only) and `railViewState()` (empty/list/detail) with unit tests.
- Extracted `PoiDetailContent` shared by mobile `PoiSheet` and desktop `ResortMapDetailRail`; desktop modal overlay removed (`md:hidden` on sheet).
- Desktop rail is **collapsible**: full-width map on All; 300ms grid-column transition shrinks map + slides rail in on chip/pin; reverses on All/clear.
- Added `desktopRailOpen()` helper; removed persistent blank rail on All.
- Mobile: chip-gated below-map list; bottom sheet on pin/list tap; map stays full-width static.
- CSS cascade fix: removed `grid-cols-1`; `.map-rail-layout.grid[data-rail-open]` owns desktop column animation.
- Chip change clears selected place. Lint, build, and unit tests pass. PO manual QA sign-off.

### File List

- `_powri/implementation-artifacts/Epic-11/11-9-poi-detail-ux-desktop-split-view.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/app/globals.css`
- `web/src/lib/map/mergePlaces.ts`
- `web/src/lib/map/mergePlaces.test.ts`
- `web/src/lib/map/railViewState.ts`
- `web/src/lib/map/railViewState.test.ts`
- `web/src/components/map/PoiDetailContent.tsx`
- `web/src/components/map/PoiSheet.tsx`
- `web/src/components/map/PoiList.tsx`
- `web/src/components/map/ResortMapDetailRail.tsx`
- `web/src/components/map/ResortMapSection.tsx`
- `web/src/components/map/ResortMapboxMap.tsx`
