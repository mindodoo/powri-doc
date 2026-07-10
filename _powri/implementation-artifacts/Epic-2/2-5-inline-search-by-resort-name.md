# Story 2.5: Inline Search by Resort Name

Status: done

baseline_commit: NO_VCS

## Story

As a **user**,
I want to search resorts by name from the top bar,
So that I can jump directly to a destination I already have in mind.

## Acceptance Criteria

1. **Given** Home sticky top bar with search icon  
   **When** I tap the search icon  
   **Then** an inline search bar slides down with smooth animation (UX-DR16)

2. **Given** search is open  
   **When** I type  
   **Then** results update live, matching resort names (English + Japanese)

3. **Given** a search with results  
   **When** query stabilises  
   **Then** `search_performed` fires with `results_count`; `query_text` only when `consent_state=granted` (scrubbed)

4. **Given** a search result  
   **When** tapped  
   **Then** navigates to `/resorts/[slug]` and fires `resort_card_tapped`

## Tasks / Subtasks

- [x] **Search matcher + types** (AC: 2)
  - [x] `web/src/lib/resort/searchResorts.ts` — client-side name/slug match
  - [x] `web/src/lib/content/resortSearchItem.ts` — serializable search index from `Resort`

- [x] **Search analytics** (AC: 3)
  - [x] `web/src/lib/analytics/trackSearch.ts` — `search_performed` with consent-gated `query_text`

- [x] **Home search UI** (AC: 1, 2, 4)
  - [x] `web/src/components/home/HomeSearchPanel.tsx` — slide-down input + live results
  - [x] `web/src/components/home/SearchResultRow.tsx` — compact result row
  - [x] Update `HomeTopBar` — enable search icon, toggle panel, solid header when open

- [x] **Wire Home page** (AC: 1–4)
  - [x] Pass search index to `HomeTopBar`

- [x] **i18n + validate**
  - [x] Search strings in `messages/en.json`
  - [x] `npm run build` + `npm run lint` pass

## Dev Notes

### UX (UX-DR16 / EXPERIENCE.md)

- Search icon in fixed Home top bar; tap expands inline panel below header (grid-row slide, 300ms)
- Header switches to white surface while search open (legibility over hero)
- Empty query: prompt copy; zero matches: mountain icon + message
- Explore search deferred — AC scopes Home only; Explore top bar in Story 2.6

### Analytics

- `search_performed`: debounced 400ms after typing; dedicated helper bypasses generic `query_text` block in `track()`
- `query_text` included only when `consent_state=granted`; scrubbed (100 char cap, email/phone redaction)
- `resort_card_tapped`: `list_context: home`, `position` from result index

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Home top bar search icon enabled; tap toggles slide-down panel with 300ms grid-row animation
- Live client-side match on `name_en`, `name_jp`, and slug; debounced `search_performed` analytics
- Result tap navigates to resort detail and fires `resort_card_tapped`
- Fresh search session on each open (key remount) clears prior query without effect setState lint issue

### File List

- `web/messages/en.json` (modified — `search` namespace)
- `web/src/lib/content/index.ts` (modified — export search items)
- `web/src/lib/content/resortSearchItem.ts` (new)
- `web/src/lib/resort/searchResorts.ts` (new)
- `web/src/lib/analytics/trackSearch.ts` (new)
- `web/src/lib/analytics/index.ts` (modified — export `trackSearchPerformed`)
- `web/src/components/home/HomeTopBar.tsx` (modified — search toggle + panel)
- `web/src/components/home/HomeSearchPanel.tsx` (new)
- `web/src/components/home/SearchResultRow.tsx` (new)
- `web/src/app/[locale]/page.tsx` (modified — pass search index)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve

### Summary

All four ACs satisfied. Home search expands inline with animation, live name matching, consent-aware analytics, and result navigation with `resort_card_tapped`. Build and lint pass.

### Review Findings

- [x] [Review][Defer] Explore tab has no search icon yet — Story 2.6 (EXPERIENCE.md lists both Home and Explore)
- [x] [Review][Defer] `list_context` for search taps is `home` — tracking plan enum has no `search`; consider adding in tracking-plan.md if search attribution needed
- [x] [Review][Defer] `/api/search` stub unused — client-side Phase 1 is correct per architecture
- [x] [Review][Note] `search_performed` fires on debounced typing only (not on open/close) — matches AC intent
