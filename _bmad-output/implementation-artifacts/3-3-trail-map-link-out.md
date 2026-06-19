# Story 3.3: Trail Map Link-Out

Status: done

baseline_commit: 48d7544

## Story

As a **user who wants terrain detail**,
I want a clear link to the official trail map,
So that I can study runs on the resort's authoritative map.

## Acceptance Criteria

1. **Given** a resort with `trail_map_url` in frontmatter  
   **When** I view the detail page  
   **Then** a trail map section appears after Terrain & Mountain Data

2. **Given** the trail map link  
   **When** tapped  
   **Then** official map opens in a new tab with `rel="noopener noreferrer"` (FR-2, NFR-12)

3. **Given** a trail map tap  
   **When** the link opens  
   **Then** `trail_map_opened` fires with `resort_slug`

4. **Given** a trail map tap  
   **When** the link opens  
   **Then** `external_link_clicked` fires with `link_type: trail_map` and `destination_domain`

5. **Given** no `trail_map_url`  
   **When** the detail page loads  
   **Then** the trail map section is omitted

## Tasks / Subtasks

- [x] **TrailMapSection component** (AC: 1, 2, 5)
  - [x] SectionHeader + optional `trail_map_note` + CTA link

- [x] **Analytics** (AC: 3, 4)
  - [x] `trail_map_opened` + `external_link_clicked` on tap
  - [x] `destination_domain` from URL hostname

- [x] **Wire into detail page** (AC: 1)
  - [x] Extend sections mapper; render after terrain in `ResortDetailContent`
  - [x] i18n strings

- [x] **Validate**
  - [x] `npm run build` + `npm run lint`

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Trail map section renders after terrain when `trail_map_url` is set (all 20 published resorts have URLs)
- Optional `trail_map_note` shown above CTA button
- Tap fires both analytics events with parsed `destination_domain`

### File List

- `web/messages/en.json` (modified — `trailMap`, `viewTrailMap`)
- `web/src/lib/analytics/externalLink.ts` (new)
- `web/src/lib/resort/detailSections.ts` (modified — trail map fields)
- `web/src/components/resort/TrailMapSection.tsx` (new)
- `web/src/components/resort/ResortDetailContent.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All five ACs satisfied. Trail map section appears in correct order, opens cleanly in new tab, omits when URL absent, and fires both tracking events on tap.

### Review Findings

- [x] [Review][Note] Analytics fire on click before navigation — standard pattern; link still opens via native `<a>` behavior
- [x] [Review][Defer] Getting There section still absent — Story 3.4
- [x] [Review][Pass] All published resorts include `trail_map_url` in content
