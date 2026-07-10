# Story 2.4: Filter Chips & Instant List Filtering

Status: done

baseline_commit: NO_VCS

## Story

As a **user**,
I want to filter resorts by region and attributes instantly,
So that I can narrow 20 destinations without waiting for page loads.

## Acceptance Criteria

1. **Given** the Home or Explore resort list  
   **When** I tap a filter chip (All · Hokkaido · Nagano · Niigata · Tohoku · Beginner Friendly · Deep Powder · Family)  
   **Then** the list filters client-side with no full page reload (FR-1, UX-DR6)

2. **Given** filter chips  
   **When** one is selected  
   **Then** single-select behaviour applies; active chip shows active state (accent fill + semibold per DESIGN.md)

3. **Given** a filter selection  
   **When** applied  
   **Then** `filter_applied` fires with `filter_type`, `filter_value`, `results_count`

4. **Given** a filter with zero matches  
   **When** applied  
   **Then** `filter_zero_results` fires (NFR-13) and empty state shows mountain icon + copy per EXPERIENCE.md

## Tasks / Subtasks

- [x] **Filter model + matcher** (AC: 1)
  - [x] `web/src/lib/resort/filters.ts` — chip definitions, region/attribute predicates on `ResortListItem`
  - [x] Extend `ResortListItem` with `bestFor[]` for attribute matching

- [x] **FilterChip + FilterChipRow** (AC: 1, 2)
  - [x] `web/src/components/ui/FilterChip.tsx` — active/inactive tokens, 150ms tap scale
  - [x] `web/src/components/resort/FilterChipRow.tsx` — horizontal scroll, single-select

- [x] **DiscoveryList client wrapper** (AC: 1, 3, 4)
  - [x] `web/src/components/resort/DiscoveryList.tsx` — filter state, analytics, empty state
  - [x] Update `ResortList` — `filtersActive`, `list_context: filtered`, reset pagination on filter change

- [x] **Wire Home + Explore** (AC: 1)
  - [x] Home: chips between discovery heading and list
  - [x] Explore: replace placeholder with same `DiscoveryList` + resorts data

- [x] **i18n + validate**
  - [x] Filter labels + empty copy in `messages/en.json`
  - [x] `npm run build` + `npm run lint` pass

## Dev Notes

### Filter mapping (from content `region` + `best_for`)

| Chip | Match rule | Count (20 resorts) |
|------|------------|-------------------|
| All | no filter | 20 |
| Hokkaido / Nagano / Niigata / Tohoku | `region` exact | 6 / 5 / 6 / 3 |
| Beginner Friendly | `best_for` contains `beginners`, `beginner`, or `all levels` | 10 |
| Deep Powder | `best_for` contains `powder` | 13 |
| Family | `best_for` contains `families` or `family` | 12 |

### Architecture

- Client-side filter over build-time resort index (architecture.md FR-1)
- Shared `DiscoveryList` for Home (`list_context: home|filtered`) and Explore (`list_context: explore|filtered`)
- Card taps pass `filters_active: [filter_value]` when filter ≠ All

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Eight single-select filter chips with horizontal scroll; instant client-side filtering, no navigation
- `FilterChip` uses Theme A active/inactive tokens + semibold active state + 150ms tap scale
- Analytics: `filter_applied` on change; `filter_zero_results` when count is 0; duplicate tap on active chip ignored
- Empty state: Lucide `Mountain` icon + i18n copy
- Explore tab now shows real filter + list (quiz CTA still Story 2.6)

### File List

- `web/messages/en.json` (modified — `filters` namespace)
- `web/src/lib/content/resortListItem.ts` (modified — `bestFor` on list item)
- `web/src/lib/resort/filters.ts` (new)
- `web/src/components/ui/FilterChip.tsx` (new)
- `web/src/components/resort/FilterChipRow.tsx` (new)
- `web/src/components/resort/DiscoveryList.tsx` (new)
- `web/src/components/resort/ResortList.tsx` (modified — `filtered` context, `filtersActive`)
- `web/src/app/[locale]/page.tsx` (modified — `DiscoveryList`)
- `web/src/app/[locale]/explore/page.tsx` (modified — `DiscoveryList`)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve

### Summary

All four ACs satisfied. Filter chips render on Home and Explore with single-select client-side filtering, correct active styling, analytics events per tracking plan, and empty state with mountain icon. Build and lint pass.

### Review Findings

- [x] [Review][Fixed] Re-tapping active chip duplicated `filter_applied` — guarded in `handleFilterChange`
- [x] [Review][Defer] Home chip row sits below "Resort discovery" heading, not immediately under hero — PRD §12.5 shows chips directly below hero; reorder in a polish pass if desired
- [x] [Review][Defer] Explore quiz CTA prominence — Story 2.6
- [x] [Review][Defer] No automated unit tests for `filterResorts()` — acceptable for Phase 1; add if filter logic grows (quiz scoring in 2.7)
