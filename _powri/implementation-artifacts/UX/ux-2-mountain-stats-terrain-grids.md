# Story UX-2: Mountain stats 3×2 + terrain split 3×1

Status: ready-for-dev

Version when done: **2.12.2.2**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a skier scanning resort detail,  
I want mountain numbers in a borderless infographic grid,  
so that elevation, runs, and terrain mix are visual — not a chip dump plus a definition list.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-2, D1, D3).

## Acceptance Criteria

1. **3×2 grid** (3 columns × 2 rows), **no cell borders**, of exactly these six fields when present:
   - top elevation (`topElevationM`)
   - base elevation (`baseElevationM`)
   - number of runs (`numRuns`)
   - longest run (`longestRunM`)
   - steepest gradient (`steepestGradientDeg`)
   - lifts (`lifts`)
2. **Do not show vertical drop** (`verticalDropM`) in this grid (D1). Schema field may remain in markdown; it is not displayed here.
3. Separate **3×1** row for `terrain_split`: labels **Beginner**, **Intermediate**, **Advanced** (i18n), **percentage underneath** (e.g. `24%`) — not a single “Beginner 24% · Intermediate …” fact line. No bars required (D3).
4. Desktop: both grids **spread proportionally** across the content column (`width: 100%` / equal columns). Mobile: same 3-column structure (compact type), not a single-column list.
5. Other terrain copy (snowfall, parks, night skiing, tree policy, course breakdown) **stays** below/as today — this story does not delete them.
6. Missing values: omit the cell or show an em dash consistently; do not invent numbers. Terrain column omitted only if that split share is empty.

## Testing & Definition of Done

- [ ] **Unit (Vitest):** `web/src/lib/resort/detailSections.ts` (or new mapper) — six stats + terrain split shape; **assert vertical drop is not in the grid payload**
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** N/A
- [ ] **Content / resorts:** `npm run test:launch`
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual desktop + mobile on a resort with full split (e.g. Furano) and one with sparse stats
- [ ] **Manual QA:** PO — proportional desktop spread, no borders

## Tasks / Subtasks

- [ ] Extend `toResortDetailSectionsData` with explicit mountain-grid + terrain-split DTOs (AC: 1–3, 6)
- [ ] New presentational component (do **not** overload `StatRow` chips) (AC: 1, 4)
- [ ] Wire in terrain section of `ResortDetailContent`; stop putting longest/steepest/split only in `terrainFacts` dl if they now live in the grid (AC: 5)
- [ ] Vitest + lint/build

## Dev Notes

**Today:** `detailSections.ts` dumps runs/vertical/area/lifts into `seasonStats` chips (`StatRow` wrap + `max-w-[28ch]`). Longest run, terrain split, steepest go to `terrainFacts` `<dl>`. `topElevationM` / `baseElevationM` are on `Resort` (`loadResorts.ts`) but **not shown** on detail. Vertical is in season chips — **remove from UI grid**, keep data.

**Reuse:** Formatters in `detailSections.ts` (`formatNumber`). i18n under `resortDetail` — add keys for grid labels (Top / Base / Runs / Longest / Steepest / Lifts) rather than embedding English in TSX.

**Do not:** New content-model fields. Change `StatRow` globally in a way that breaks season chips (lift pass season can stay chips). CSS cell `border`.

**Lifts** is a string (`resort.lifts`), not always a count — show the string, punchy if long (still no clamp that hides the value; wrap inside the cell).

### Files

| Path | Role |
|------|------|
| `web/src/lib/resort/detailSections.ts` | UPDATE mapping |
| `web/src/lib/resort/detailSections.test.ts` | NEW if none |
| `web/src/components/resort/StatRow.tsx` | Do not force 3×2 through this |
| `web/src/components/resort/MountainStatsGrid.tsx` | NEW (name flexible) |
| `web/src/components/resort/TerrainSplitGrid.tsx` | NEW or same file |
| `web/src/components/resort/ResortDetailContent.tsx` | Wire |
| `web/messages/en.json` | Labels |
| `web/src/lib/content/types.ts` | Read-only field names |

### References

- PRD UX-2, D1, D3
- `web/src/lib/content/schema.ts` `terrain_split`, `top_elevation_m`, `steepest_gradient_deg`

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
