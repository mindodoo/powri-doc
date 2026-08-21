---
baseline_commit: d20bc82b758ea6b60d729668f613016ded7b0675
---

# Story UX-2: Mountain stats metric table + course-level bar

Status: done

Version when done: **2.12.2.2**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a skier scanning resort detail,  
I want mountain numbers in a borderless infographic grid,  
so that elevation, runs, and terrain mix are visual — not a chip dump plus a definition list.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-2, D1, D3).

**UX lock (2026-08-20):** Presentation evolved from “3×2 cards + 3×1 percentage boxes” to a **responsive hairline metric table** plus a **Course level stacked bar** with Japan trail-map markers. Data mapping from the first pass is kept; visual contract below supersedes original AC 3–4 and D3’s “no bars required” (bars are now required). D1 (no vertical drop) still stands.

## Acceptance Criteria

1. **Mountain stats** — exactly these six fields when present, in this order: Top · Base · Steepest / Runs · Longest · Lifts:
   - top elevation (`topElevationM`)
   - base elevation (`baseElevationM`)
   - steepest gradient (`steepestGradientDeg`)
   - number of runs (`numRuns`)
   - longest run (`longestRunM`)
   - lifts (`lifts`)
2. **Do not show vertical drop** (`verticalDropM`) in this grid (D1). Schema field may remain in markdown; it is not displayed here.
3. **Course level** block for `terrain_split` (not a fact-line, not three equal cards):
   - `h3` title **Course level** (i18n). No extra icon beside the title.
   - Full-width **stacked bar** (beginner / intermediate / advanced).
   - Legend under the bar: Japan trail markers + label + percentage (e.g. `40%`).
   - Markers: **beginner = green circle**, **intermediate = red square**, **advanced = black diamond**. Markers live in the legend only — not inside the bar.
4. **Responsive mountain stats:** **2 columns** below `md` (768px / 48rem); **3 columns** at `md` and up. Both breakpoints `width: 100%` / equal columns. Not a single-column list. Course bar is full width at every breakpoint.
5. Other terrain copy (snowfall, parks, night skiing, tree policy, **By ability level** / `courseBreakdown`) **stays** below — this story does not delete them. Course level sits **above** those facts, **below** mountain stats. **Follow-up (2026-08-20)** replaces that leftover stack — see **Follow-up: leftover Terrain stats** below. Do not treat the follow-up as in-scope for UX-2 review.
6. Missing values: **omit** the cell / split share; do not invent numbers or empty coloured slots. Do not use em dashes as placeholders.

### Visual (locked)

7. **No cards:** no `bg-surface` tiles, no drop shadows, no `rounded-card` on mountain-stat cells or course-level segments.
8. Mountain stats = borderless **hairline table**: inner `1px solid {colors.divider}` only (no outer frame around the whole table). Icon 14px Lucide / stroke 2 / `{colors.text-secondary}` + value `{typography.body}` `{colors.text-primary}` + label `{typography.badge}` `{colors.text-secondary}`. Centre-aligned. `lifts` wraps; do not truncate.
9. Course bar: segment width = parsed percent (`normalizeTerrainPercent`; do **not** renormalise so 40/40 becomes 50/50). Unfilled remainder is `{colors.divider}` track. 2px `{colors.bg}` gutter between segments. Omit empty shares. No fill animation.
10. Colour is never the only difficulty signal: shape + colour + label + %. Do not use `{colors.error}`, `{colors.accent}`, or Tailwind `green-*` / `red-*` for difficulty. Use the course tokens below.
11. Legend: one row; each item **spans the same width as its bar segment** (not equal 1/n columns) so the stack is centred between that category’s start and end. Present shares only. Per item: marker (top) → label `{typography.badge}` → percentage `{typography.caption}` `{colors.text-primary}`. Markers `aria-hidden`; group labelled by **Course level**. PO 2026-08-20: align to bar geometry, not equal flex.

### Tokens (Theme A)

Reuse existing Theme A tokens unless marked **new**. Add new CSS variables in `web/src/styles/tokens.css` (Theme A) — do not hard-code one-off hex in the component.

| Token | Value | Use |
|---|---|---|
| `{colors.bg}` | `#F7F5F2` | Page; bar gutters |
| `{colors.divider}` | `#E8E5E0` | Metric inner rules; bar track |
| `{colors.text-primary}` | `#1C1A17` | Stat values; legend % |
| `{colors.text-secondary}` | `#6E6A63` | Stat labels; Course level title; legend names |
| `{typography.body}` | 15px / 400 / 1.6 DM Sans | Stat values |
| `{typography.badge}` | 11px / 500 / 0.08em uppercase | Stat labels; Course level; legend names |
| `{typography.caption}` | 12px / 400 / 1.4 | Legend percentages |
| **`{colors.course-beginner}`** *(new)* | `#3F7A4E` | Bar + circle fill |
| **`{colors.course-intermediate}`** *(new)* | `#C45C4A` | Bar + square fill |
| **`{colors.course-advanced}`** *(new)* | `#1C1A17` | Bar + diamond fill |
| **`{components.course-bar.height}`** *(new)* | **10px** | All breakpoints |
| **`{components.course-bar.radius}`** *(new)* | **4px** | Not pill / `{rounded.badge}` |
| **`{components.course-bar.gutter}`** *(new)* | **2px** `{colors.bg}` | Between segments |
| **`{components.course-bar.min-segment}`** *(new)* | **4px** | Tiny shares still visible |
| **`{components.course-marker.size}`** *(new)* | **10px** | Bounding box: circle, square (≤1px radius), diamond (10×10 square rotated 45°, overflow visible) |

### Spacing (8px grid)

Keep Terrain `SectionHeader` → content at **12px** (`mt-3`).

| Gap | Size |
|---|---|
| Mountain stats → Course level block | **24px** (`{spacing.3}`) |
| Course level title → bar | **8px** (`{spacing.1}`) |
| Bar → legend | **8px** (`{spacing.1}`) |
| Course level block → remaining terrain facts / By ability level | **24px** (`{spacing.3}`) |
| Legend column gap | Match bar gutters (**2px**) so columns share the bar’s start/end |
| Marker → legend copy | **6px** (only 6px exception — avoid early wrap at 375) |
| Mountain-stat grid gap | **0** (hairlines only) |
| Cell padding &lt; `md` | **12px** vertical, **8px** horizontal |
| Cell padding ≥ `md` | **16px** vertical, **8px** horizontal |

### Out of scope / do not

- Pie / donut / three independent progress rows.
- Shapes drawn inside the bar.
- Title icon beside **Course level**.
- 3-col mountain stats on mobile; 2-col on desktop.
- New content-model fields.
- Changing `StatRow` globally (season snapshot chips stay).
- Leftover Terrain facts listed in **Follow-up: leftover Terrain stats** (out of scope until that follow-up is pulled).

## Follow-up: leftover Terrain stats

**Status:** first pass implemented (Sally + PO 2026-08-20). **Revision specified** (Sally + PO 2026-08-21) — module titles, snow-quality eyebrow, parks/night icons. Ready for a small implementation pass.  
**Does not change** UX-2 AC 1–11. Do **not** add these fields into the locked six-cell mountain table. Do **not** treat this revision as in-scope for UX-2 review of AC 1–11.

Replace chips + definition-list dump under Course level with modules that match the hairline table / course-bar language. Same Theme A contract: no cards, no shadows, no `rounded-card` on metric cells.

### Stack (top → bottom)

Keep Terrain `SectionHeader` → content at **12px**. **24px** (`{spacing.3}`) between modules except where noted.

1. Mountain stats (UX-2, unchanged) — **no module title** (PO 2026-08-21). Six labelled cells under **Terrain** are enough.
2. **Snow & season** — `h3` title + 2-cell hairline row
3. **Course level** — bar + legend (UX-2) **then** course notes (this follow-up), one labelled group
4. **Parks & night** — `h3` title + 2-cell hairline
5. **Tree runs & off-piste** — one labelled prose row (`dt`/`dd`)

### Module titles (revision 2026-08-21)

**Snow & season** and **Parks & night** use the same inner title as **Course level** — not a second `SectionHeader` (that would compete with **Terrain**).

- Element: `h3`. Type: `{typography.badge}` `{colors.text-secondary}`. No Lucide beside the title.
- i18n (under `resortDetail`): `snowAndSeason` = `Snow & season`; `parksAndNight` = `Parks & night`.
- Title → grid (or quality-only caption): **8px** (`{spacing.1}`), same as Course level title → bar.
- Wrap each module in `role="group"` labelled by that `h3`.
- If only one cell (or quality-only, no cells) is present: **still show the title**. Omit the whole module only when there is nothing to show.
- **Do not** title mountain stats. **Do not** use **Conditions** / **Condition** — that word means a live snow report (base depth, overnight); this data is typical season + annual average.

### Snow & season

Hairline table, same recipe as mountain stats (icon 14px Lucide / stroke 2 / `{colors.text-secondary}` + value `{typography.body}` `{colors.text-primary}` + label `{typography.badge}` `{colors.text-secondary}`). Centre-aligned. Inner hairlines only. **2 columns** at every breakpoint (`season` beside `snowfall`). Omit empties (including placeholder `"~"`). Wrap like `lifts`; do not truncate.

| Cell | Field | Label | Icon |
|---|---|---|---|
| Snowfall | `avg_annual_snowfall_m` | Annual snowfall | `Snowflake` |
| Season | `season_typical` | Season | `CalendarRange` |

**Snow quality is not a stat cell and not a chip.** `snow_quality` is an **eyebrow caption above the snowfall value** — character of the snow, not a third metric. The metres stay the only `{typography.body}` `{colors.text-primary}` line in the cell.

Snowfall cell stack (top → bottom):

1. `Snowflake`
2. Quality eyebrow (if present)
3. Amount (`avg_annual_snowfall_m`)
4. Label **Annual snowfall**

- Type: `{typography.caption}` `{colors.text-secondary}`, centre-aligned, wraps.
- Placement: inside the snowfall cell, **above** the numeric value (below the icon). Gap **4px** eyebrow → value (half-step; keep the cell from growing a full 8px twice). No extra gap when quality is omitted (icon → value → label, same as other metric cells).
- **PO lock:** do not give snow quality its own cell, `CloudSnow` icon, or `StatRow` pill. Do not promote quality to body type.
- If snowfall is present and quality is missing: omit the eyebrow.
- If quality is present and snowfall is omitted: **do not** invent a snowfall number. Show quality as a **full-width caption** under the snow & season row (still caption type; no fake cell). If season is also omitted, title + that caption is the whole module.
- Stop routing snowfall / snow quality through `addChipOrTerrainFact` / Terrain `StatRow`.

### Course notes (after the bar)

`courseBreakdown` belongs **immediately under the Course level legend**, inside the same `role="group"` labelled **Course level**. Drop the competing **By ability level** `h3`.

Parse existing markdown headings:

`**Beginners.**` · `**Intermediates.**` · `**Advanced / experts.**` (and close variants)

Render **three stacked full-width rows** (not columns aligned to bar widths — a 20% advanced column would crush the expert paragraph):

- Japan trail **marker** (same tokens / size as the legend: circle / square / diamond) + level label `{typography.badge}` `{colors.text-secondary}` on one line; body `{typography.body}` `{colors.text-primary}` left-aligned underneath.
- Strip the `**Beginners.**` (etc.) prefix from the paragraph; marker + label replace it.
- No extra Lucide on these rows. No second stacked bar. No accordion.
- **Legend → first note: 16px.** **Between notes: 12px.** Then **24px** before parks + night.
- Omit a level if that heading is absent. If parsing fails, fall back to today’s pre-line block — do not invent structure.

### Parks & night

`h3` title **Parks & night** (see **Module titles**). Cell labels stay **Terrain parks** / **Night skiing**.

| Cell | Field | Label | Icon |
|---|---|---|---|
| Parks | `terrain_parks` | Terrain parks | `Cone` |
| Night | `night_skiing` | Night skiing | `Moon` |

Same hairline cells as mountain stats (icon 14px Lucide / stroke 2 / `{colors.text-secondary}`; label under wrapping value). **2 columns at every breakpoint.** Omit if missing. If only one cell remains, it still shows its icon.

**Do not** use `Trees` / `TreePine` for parks — that collides with the **Tree runs & off-piste** row immediately below. Parks in this content are snow parks (rails / kickers / named parks), not tree skiing.

### Tree runs

`tree_runs_offpiste_policy` stays **one labelled prose fact** under parks + night. It is policy copy, not a metric. No icon. No hairline cell.

### Out of scope for the follow-up

- New content-model fields; `best_months` / `peak_avoid` stay off this section.
- Expanding the six mountain-stat cells.
- Changing `StatRow` globally (Season snapshot chips stay).
- Title icon beside **Course level**, **Snow & season**, or **Parks & night**.
- Module title on mountain stats.
- Title copy **Conditions** / **Condition**.
- `Trees` / `TreePine` (or any tree glyph) on the parks cell.

## Testing & Definition of Done

- [x] **Unit (Vitest):** `web/src/lib/resort/detailSections.ts` (or new mapper) — six stats + terrain split shape; **assert vertical drop is not in the grid payload**
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A
- [x] **Content / resorts:** `npm run test:launch`
- [x] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual desktop + mobile on a resort with full split (e.g. Furano) and one with sparse stats
- [x] **Manual QA:** PO — 2-col mobile / 3-col desktop stats; hairlines not cards; Course level bar + trail markers; no shadows
- [x] **Follow-up revision (2026-08-21):** Snow quality eyebrow above metres; **Snow & season** / **Parks & night** titles; `Cone` + `Moon`; mountain stats still untitled

## Tasks / Subtasks

- [x] Extend `toResortDetailSectionsData` with explicit mountain-grid + terrain-split DTOs (AC: 1–3, 6)
- [x] New presentational component (do **not** overload `StatRow` chips) (AC: 1, 4)
- [x] Wire in terrain section of `ResortDetailContent`; stop putting longest/steepest/split only in `terrainFacts` dl if they now live in the grid (AC: 5)
- [x] Season snapshot: replace metric chips with all formatted `best_for` tags; keep lift pass + last reviewed (PO clarification)
- [x] Vitest + lint/build
- [x] **Visual revision:** Theme A course colour tokens in `tokens.css` (AC: 10)
- [x] **Visual revision:** `MountainStatsGrid` — hairline table, 2-col → 3-col at `md`, no cards/shadows (AC: 4, 7–8)
- [x] **Visual revision:** `TerrainSplitGrid` — Course level title, stacked bar, trail-marker legend (AC: 3, 9–11)
- [x] **Visual revision:** i18n `Course level`; Terrain inner gaps 24px between modules (spacing table)
- [x] Re-run lint / build / unit; manual Furano + sparse resort
- [x] **Follow-up leftover Terrain:** snow & season 2-cell hairline; snow quality as caption (not a cell); omit `~`
- [x] **Follow-up leftover Terrain:** parse course notes under Course level group; drop **By ability level** `h3`
- [x] **Follow-up leftover Terrain:** parks + night 2-cell hairline (no icons); tree policy labelled prose
- [x] **Follow-up leftover Terrain:** stop routing snowfall / quality / season / parks / night through Terrain `StatRow` / leftover `dl`
- [x] **Follow-up leftover Terrain:** Vitest + lint/build
- [x] **Follow-up revision 2026-08-21:** snow quality caption **above** the metres (eyebrow); omit empty gap when quality missing
- [x] **Follow-up revision 2026-08-21:** `h3` **Snow & season** + **Parks & night** (Course level type; 8px to grid; labelled groups); i18n keys; mountain stats stay untitled
- [x] **Follow-up revision 2026-08-21:** parks `Cone` + night `Moon` (14px / stroke 2); never tree glyphs on parks
- [x] **Follow-up revision 2026-08-21:** lint / build / unit; manual Furano + sparse (quality-only, one-cell parks)

### Review Findings

- [x] [Review][Dismiss] Content shortened `lifts` / `season_typical` instead of wrapping — PO 2026-08-21: keep compact counts/ranges; fields are stats, not descriptive text.
- [x] [Review][Patch] Range terrain splits parsed as a single invented percent [`web/src/lib/resort/detailSections.ts:135`]
- [x] [Review][Patch] Course-level legend overflows when segment `%` + `min-width` + `shrink-0` exceed 100% [`web/src/components/resort/TerrainSplitGrid.tsx:70`]

## Dev Notes

**Shipped (first pass):** `mountainStats` + `terrainSplitColumns` DTOs; `normalizeTerrainPercent()`; grids wired in Terrain; longest/steepest/split removed from `terrainFacts`; vertical drop not on detail; season snapshot = all `best_for` chips.

**Current UI debt:** `MountainStatsGrid` is `grid-cols-2` cards with box shadow on all breakpoints. `TerrainSplitGrid` is three tinted shadowed boxes (green/red/neutral). Replace per Visual lock — do not polish the cards.

**Reuse:** Existing Lucide map in `MountainStatsGrid`. `normalizeTerrainPercent` for bar widths. i18n under `resortDetail` — add `courseLevel`, `snowAndSeason`, `parksAndNight`; keep `terrainSplit.beginner|intermediate|advanced`. `MetricHairlineTable` already supports `caption`; revision only changes **order** (eyebrow above value) plus module titles on `SnowSeasonGrid` / `ParksNightGrid`.

**Lifts** is a string (`resort.lifts`), not always a count — wrap inside the cell; no clamp that hides the value.

**A11y:** Course level, Snow & season, and Parks & night are labelled groups. Markers decorative. Bar may be `aria-hidden` if the legend lists the same data.

### Files

| Path | Role |
|------|------|
| `web/src/styles/tokens.css` | ADD `--color-course-*` (+ optional bar/marker vars) |
| `web/src/components/resort/MountainStatsGrid.tsx` | UPDATE — hairline table, responsive cols |
| `web/src/components/resort/TerrainSplitGrid.tsx` | UPDATE — title, bar, markers |
| `web/src/components/resort/ResortDetailContent.tsx` | UPDATE — title wiring / 24px module gaps if not internal to child |
| `web/messages/en.json` | ADD `courseLevel`; revision: ADD `snowAndSeason`, `parksAndNight` |
| `web/src/components/resort/SnowSeasonGrid.tsx` | UPDATE — module title + quality eyebrow above value |
| `web/src/components/resort/ParksNightGrid.tsx` | UPDATE — module title; `Cone` / `Moon` |
| `web/src/components/resort/MetricHairlineTable.tsx` | UPDATE — caption (eyebrow) **above** the value when present |
| `web/src/lib/resort/detailSections.ts` | Keep mapping unless bar needs numeric percent alongside display string |
| `web/src/lib/content/types.ts` | Read-only field names |

### References

- PRD UX-2, D1, D3 (D3 “no bars required” superseded by this lock)
- Theme A: [`ux-designs/ux-phase1/design.md`](../../planning-artifacts/ux-designs/ux-phase1/design.md) — borders over shadows; no hard shadows on editorial blocks
- `web/src/lib/content/schema.ts` `terrain_split`, `top_elevation_m`, `steepest_gradient_deg`

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (dev-story)

### Debug Log References

- PO clarifications (first pass): omit missing cells; grid order Row1 Top/Base/Steepest Row2 Runs/Longest/Lifts; normalize terrain %; season snapshot → all `best_for` tags + lift pass/last reviewed.
- UX lock (Sally, 2026-08-20): 2-col mobile / 3-col desktop metric table; no shadow cards; Course level stacked bar; JP trail markers in legend; tokens/spacing as above.
- Follow-up leftover Terrain (Sally + PO 2026-08-20): snow quality = caption under snowfall, not a cell; course notes after the bar; parks+night hairline; tree policy stays prose.
- Follow-up implemented 2026-08-20: `parseCourseNotes` + `metricTwoColHairlineClass`; Terrain stack = mountain → snow & season → course level (bar + notes) → parks/night → tree prose.
- Follow-up revision (Sally + PO 2026-08-21): quality eyebrow **above** metres; titles **Snow & season** / **Parks & night**; parks `Cone` + night `Moon`; mountain stats untitled; reject **Conditions** and tree icons on parks.
- Follow-up revision implemented 2026-08-21: `MetricHairlineTable` caption-above-value; module `h3` groups; `Cone`/`Moon`; i18n `snowAndSeason` / `parksAndNight`. Dev `npm run dev` may need a restart to pick up new message keys (Turbopack can keep a stale `en.json` chunk).

### Completion Notes List

- Added `mountainStats` + `terrainSplitColumns` DTOs; `normalizeTerrainPercent()` strips `~` → clean `40%`.
- Grid order per PO: Top · Base · Steepest / Runs · Longest · Lifts; missing cells omitted.
- `MountainStatsGrid` + `TerrainSplitGrid` first pass: equal-column grids (later revised to hairline table + bar — see Visual revision tasks).
- Removed longest/steepest/split from `terrainFacts`; vertical drop not rendered anywhere on detail.
- Season snapshot shows formatted `best_for` chips (all entries) via existing `StatRow`; lift pass + last reviewed unchanged.
- Exported `formatBadge` from `resortListItem.ts` for reuse.
- Visual lock: Theme A `--color-course-*` + bar/marker vars; `MountainStatsGrid` inner hairlines (`metricCellHairlineClass`), 2-col / `md:` 3-col, no cards/shadows; `TerrainSplitGrid` Course level stacked bar (raw percents, divider remainder, 2px `bg` gutters) + JP trail-marker legend.
- `terrainSplitColumns` now includes numeric `percent` for bar width; omit unparseable shares.
- Terrain module gaps `--space-3` (24px) between stats, course level, remaining facts.
- Follow-up leftover Terrain: `snowSeason` / `parksNight` / `treeRunsPolicy` / `parseCourseNotes()`; snow quality is a caption (cell or full-width under the row); `~` snowfall omitted; Course notes live in the Course level group (no **By ability level** heading); parks+night 2-col hairline with no icons; tree policy stays labelled prose.
- Shared `MetricHairlineTable` + `CourseMarker`.
- `npm run lint && npm run build && npm run test:unit` pass.
- Follow-up revision 2026-08-21: snow quality eyebrow above metres (4px gap only when present); **Snow & season** / **Parks & night** `h3` groups (8px to grid); mountain stats stay untitled; parks `Cone` + night `Moon`; quality-only module still titled; GALA `~` snowfall omitted with full-width quality caption.

### File List

- `web/src/lib/resort/detailSections.ts` (modified)
- `web/src/lib/resort/detailSections.test.ts` (new)
- `web/src/lib/content/resortListItem.ts` (modified — export `formatBadge`)
- `web/src/components/resort/MountainStatsGrid.tsx` (new)
- `web/src/components/resort/TerrainSplitGrid.tsx` (new)
- `web/src/components/resort/ResortDetailContent.tsx` (modified)
- `web/messages/en.json` (modified — `courseLevel`, `snowAndSeason`, `parksAndNight`)
- `web/src/styles/tokens.css` (modified — course + bar tokens)
- `web/src/app/globals.css` (modified — course colours in `@theme`)
- `web/src/components/resort/MetricHairlineTable.tsx` (modified — quality eyebrow above value)
- `web/src/components/resort/CourseMarker.tsx` (new — JP trail markers)
- `web/src/components/resort/SnowSeasonGrid.tsx` (modified — module title + labelled group)
- `web/src/components/resort/ParksNightGrid.tsx` (modified — module title; `Cone` / `Moon`)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)

## Change Log

- 2026-08-14: UX-2 mountain stats 3×2 + terrain split 3×1; season snapshot best_for tags (PO clarification).
- 2026-08-20: Visual lock — responsive hairline metric table (2-col mobile / 3-col desktop); Course level stacked bar + JP trail markers; Theme A course tokens and spacing; story returned to ready-for-dev for implementation pass.
- 2026-08-20: Implemented visual lock (hairline table, Course level bar + markers, tokens, 24px module gaps).
- 2026-08-20: PO — Course level legend columns match bar segment widths (centred on each category), not equal flex.
- 2026-08-20: Follow-up specified — leftover Terrain stats (snow & season, course notes, parks/night, tree policy). Snow quality locked as caption under snowfall. Not in UX-2 review scope.
- 2026-08-20: Implemented leftover Terrain follow-up — snow & season hairline + quality caption, course notes under Course level, parks+night hairline, tree policy prose.
- 2026-08-21: PO lock — snow quality eyebrow above metres; module titles **Snow & season** and **Parks & night** (Course level `h3` recipe); parks `Cone` + night `Moon`; mountain stats stay untitled. Supersedes 2026-08-20 quality-under-value and no-icon parks/night.
- 2026-08-21: Implemented follow-up revision — quality eyebrow, module titles, parks/night icons; lint/build/unit pass.
- 2026-08-21: Code review — omit dashed terrain-split ranges (AC 6/9); legend `overflow-hidden` + `min-w-0` (no `shrink-0` overflow); compact lifts/season kept as stats.
