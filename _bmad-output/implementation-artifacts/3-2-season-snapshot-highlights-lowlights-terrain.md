# Story 3.2: Season Snapshot, Highlights, Lowlights & Terrain

Status: done

baseline_commit: 48d7544

## Story

As a **user researching a resort**,
I want scannable pros/cons and mountain stats with honest season labels,
So that I can judge fit without hype.

## Acceptance Criteria

1. **Given** a published resort  
   **When** I scroll below the hero  
   **Then** sections appear in order: Season snapshot → Highlights & Lowlights → Terrain & Mountain Data (Trail map + Getting There follow in 3.3–3.4)

2. **Given** the highlights section  
   **When** rendered  
   **Then** `HighlightLowlightRow` shows 50/50 columns with tinted backgrounds (UX-DR8)

3. **Given** season snapshot  
   **When** rendered  
   **Then** `StatRow` shows runs, vertical, area, lifts, and typical season (UX-DR9)

4. **Given** T2 pricing fields  
   **When** displayed  
   **Then** explicit season label (e.g. "¥X — 2025/26 season") and last-reviewed signal when present (NFR-6)

5. **Given** each section  
   **When** rendered  
   **Then** `SectionHeader` with divider rule (UX-DR15)

## Tasks / Subtasks

- [x] **UI components** (AC: 2, 3, 5)
  - [x] `SectionHeader`, `StatRow`, `HighlightLowlightRow`

- [x] **Content mappers** (AC: 3, 4)
  - [x] `parseBulletList()` for highlights/lowlights body markdown
  - [x] Season label + lift pass T2 line; terrain fact rows

- [x] **ResortDetailContent** (AC: 1)
  - [x] Wire into `resorts/[slug]/page.tsx` below hero
  - [x] i18n strings

- [x] **Validate**
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- Highlights/lowlights from `resort.body.highlights` / `watchOutFor` (markdown `-` bullets)
- Season stats from frontmatter: `numRuns`, `verticalDropM`, `skiableAreaHa`, `lifts`, `seasonTypical`
- T2: `liftPassAdultDay` + season label derived from `seasonConfirmed`
- `lastReviewed` from frontmatter when set
- Terrain: terrain split, snowfall, snow quality, course breakdown, night skiing, parks, tree policy
- Out of scope: Trail map link (3.3), Getting There (3.4–3.5)

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- White scroll content below hero with three sections matching mockup order
- Bullet lists parsed from markdown body; 50/50 highlight/lowlight columns with theme tints
- Season snapshot StatRow + lift pass T2 line with derived season label (e.g. 2025/26)
- Terrain section: extra stat chips, fact rows, course breakdown by ability

### File List

- `web/messages/en.json` (modified — `resortDetail.*`)
- `web/src/lib/content/parseBulletList.ts` (new)
- `web/src/lib/resort/detailSections.ts` (new)
- `web/src/components/ui/SectionHeader.tsx` (new)
- `web/src/components/resort/StatRow.tsx` (new)
- `web/src/components/resort/HighlightLowlightRow.tsx` (new)
- `web/src/components/resort/ResortDetailContent.tsx` (new)
- `web/src/app/[locale]/resorts/[slug]/page.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All five ACs satisfied for Story 3.2 scope. Three scroll sections render below hero with SectionHeader dividers, StatRow chips, HighlightLowlightRow columns, T2 lift pass season labelling, and terrain facts from frontmatter + course breakdown.

### Review Findings

- [x] [Review][Defer] Trail map + Getting There sections not yet on page — Stories 3.3–3.4
- [x] [Review][Defer] `lastReviewed` empty on all 20 resorts — line hidden until editorial fills field
- [x] [Review][Defer] Highlight rows omit 16px icons per PRD — text-only matches mockup HTML; add icons in polish pass
- [x] [Review][Pass] Section order correct for 3.2; 3.3 appends trail map below terrain
