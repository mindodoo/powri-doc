# Story 2.6: Explore Tab

Status: done

baseline_commit: NO_VCS

## Story

As a **user who wants to compare options**,
I want an Explore tab that foregrounds filters and the quiz,
So that discovery feels purposeful rather than passive browsing.

## Acceptance Criteria

1. **Given** I tap Explore in bottom nav  
   **When** the Explore screen opens  
   **Then** filter chips and "Find my resort" CTA are prominent at top (FR-1, §12.5)

2. **Given** Explore  
   **When** browsing resorts  
   **Then** the same 20-resort dataset and card components as Home are reused

3. **Given** list interactions on Explore  
   **When** events fire  
   **Then** `list_context` is `explore` or `filtered` as appropriate

4. **Given** Explore vs Home  
   **When** Explore opens  
   **Then** filters are visible immediately (no hero scroll required)

## Tasks / Subtasks

- [x] **ExploreTopBar + shared search** (AC: 4)
  - [x] Extract `TopBarWithSearch`; refactor `HomeTopBar`
  - [x] `ExploreTopBar` — solid header + inline search with `listContext: explore`

- [x] **Find my resort CTA** (AC: 1)
  - [x] `FindMyResortCta` → links to `/quiz`
  - [x] Quiz route stub until Story 2.7

- [x] **Explore page layout** (AC: 1–4)
  - [x] CTA + filters + `DiscoveryList` with `listContext="explore"`
  - [x] Top padding for fixed header; no hero

- [x] **i18n + validate**
  - [x] Explore + quiz stub strings
  - [x] `npm run build` + `npm run lint` pass

## Dev Notes

- PRD §12.5: Explore shares Home data; filters + quiz CTA upfront; no separate data source
- Layout order: fixed top bar → subtitle → quiz CTA → filter chips → resort list
- `/quiz` stub — full 4-question flow in Story 2.7

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Explore page: fixed `ExploreTopBar` with search (parity with Home), prominent accent CTA, then filter chips + shared `DiscoveryList`
- Extracted `TopBarWithSearch` shared by Home and Explore; search panel accepts `listContext` for analytics
- No hero on Explore — filters visible on first paint (AC 4)
- `/quiz` stub page so CTA navigates without 404 until Story 2.7

### File List

- `web/messages/en.json` (modified — `explore`, `quiz` namespaces)
- `web/src/components/layout/TopBarWithSearch.tsx` (new)
- `web/src/components/home/HomeTopBar.tsx` (modified — uses shared top bar)
- `web/src/components/home/HomeSearchPanel.tsx` (modified — `listContext` prop)
- `web/src/components/explore/ExploreTopBar.tsx` (new)
- `web/src/components/explore/FindMyResortCta.tsx` (new)
- `web/src/app/[locale]/explore/page.tsx` (modified — full Explore layout)
- `web/src/app/[locale]/quiz/page.tsx` (new — stub)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve

### Summary

All four ACs satisfied. Explore foregrounds quiz CTA and filter chips without a hero, reuses Home data/components, and wires correct `list_context` for list and search analytics. Build and lint pass.

### Review Findings

- [x] [Review][Defer] `/quiz` is a placeholder — Story 2.7 implements full quiz + `quiz_started` / `quiz_completed`
- [x] [Review][Defer] Onboarding "Find my resort" lands on Explore (not `/quiz` directly) — acceptable; user sees CTA immediately
- [x] [Review][Note] Explore page title shown in top bar; eyebrow "Explore" retained in body for wayfinding
