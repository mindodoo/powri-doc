# Story 7.5: Terrain & Mountain Stat Chips — Shorter Tags

Status: done

## Story

As a **detail page reader**,  
I want terrain and mountain data chips to be short scannable tags,  
So that I do not need horizontal scrolling to read stats on mobile.

## Acceptance Criteria

1. No horizontal scroll on stat chip rows at 375px — **Pass**
2. Chip values ≤28 characters (editorial + code guard) — **Pass**
3. Long prose moved out of chips into terrain facts list — **Pass** (`addChipOrTerrainFact`)
4. All 20 resorts shortened — **Pass**

## Tasks / Subtasks

- [x] `StatRow`: remove `overflow-x-auto`; `flex-wrap` + `max-w-[28ch] truncate` + `title` tooltip
- [x] `detailSections.ts`: `CHIP_MAX_LENGTH = 28`; long lifts/season/snowfall/snowQuality → `terrainFacts`
- [x] i18n: `terrainFacts.lifts`, `season`, `snowfall`, `snowQuality`
- [x] Editorial pass: all 20 resort frontmatter chip fields shortened
- [x] `npm run lint` + `npm run build`

## Dev Agent Record

### Completion Notes

- Chip fields: `lifts`, `season_typical`, `avg_annual_snowfall_m`, `snow_quality` (plus formatted runs/vertical/area)
- Values still >28 chars after editorial pass fall through to labelled terrain facts (full prose preserved)
- Worst offenders shortened: Zao, Lotte Arai, Kandatsu, Hakuba, Niseko, Shiga

### File List

- `web/src/components/resort/StatRow.tsx`
- `web/src/lib/resort/detailSections.ts`
- `web/messages/en.json`
- `content/resorts/*.md` (20 resorts)
