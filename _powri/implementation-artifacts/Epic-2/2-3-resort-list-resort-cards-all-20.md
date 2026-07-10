# Story 2.3: Resort List & Resort Cards (All 20)

Status: done

baseline_commit: NO_VCS

## Story

As a **user**,
I want to scroll through all 20 Japan ski destinations as beautiful cards,
So that I can browse every option from one screen.

## Acceptance Criteria

1. **Given** 20 published resort content files  
   **When** I view Home below the hero  
   **Then** each resort renders as a `ResortCard` with image area, name, region, tagline, and 2–3 badge pills (FR-1, UX-DR5)

2. **Given** the resort list  
   **When** browsing  
   **Then** all 20 resorts are reachable — initial render shows 10 with "Show more"; ≤2 taps to see all

3. **Given** a resort card  
   **When** tapped  
   **Then** navigates to `/resorts/[slug]` and fires `resort_card_tapped` with `resort_slug`, `position`, `list_context: home`

4. **Given** card tap  
   **When** navigating to detail  
   **Then** shared-element transition is initiated via `viewTransitionName` on card image and detail hero stub (UX-DR5)

## Tasks / Subtasks

- [x] **Content unblock**
  - [x] All 20 resort files set to `content_status: published`

- [x] **Resort list mapper** (AC: 1)
  - [x] `toResortListItem()` — serializable card props from `Resort`
  - [x] Image: `card` → `hero[0]` → gradient placeholder

- [x] **BadgePill + ResortCard** (AC: 1)
  - [x] `web/src/components/ui/BadgePill.tsx`
  - [x] `web/src/components/resort/ResortCard.tsx` — 16/10 image, region, badges, chevron

- [x] **ResortList** (AC: 2, 3)
  - [x] `web/src/components/resort/ResortList.tsx` — show 10, load more, analytics

- [x] **Home page** (AC: 1–3)
  - [x] Replaced placeholder with `ResortList`

- [x] **Detail view transition stub** (AC: 4)
  - [x] Hero stub + matching `viewTransitionName`; `experimental.viewTransition` in next.config

- [x] **i18n + validate**
  - [x] `home.showMore`, `home.resortCount`, `home.noResorts`
  - [x] `npm run build` + `npm run lint` pass

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- 20 published resorts render on Home; 10 initial + "Show N more" button
- Card images use gradient placeholder (editorial images not yet in frontmatter)
- Analytics: `resort_card_tapped`, `load_more_clicked` via existing `track()`
- View Transitions enabled for shared-element naming card → detail hero

### File List

- `content/resorts/*.md` (20 files — `content_status: published`)
- `web/next.config.ts` (modified — viewTransition)
- `web/messages/en.json` (modified)
- `web/src/lib/content/resortListItem.ts` (new)
- `web/src/lib/content/index.ts` (modified)
- `web/src/components/ui/BadgePill.tsx` (new)
- `web/src/components/resort/ResortCard.tsx` (new)
- `web/src/components/resort/ResortList.tsx` (new)
- `web/src/app/[locale]/page.tsx` (modified)
- `web/src/app/[locale]/resorts/[slug]/page.tsx` (modified — hero stub + view transition)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Twenty published resorts list on Home with paginated reveal, card UI matches DESIGN spec, analytics wired, view transition names paired on card and detail hero.

### Review Findings

- [x] [Review][Defer] No resort card images in content yet — gradient placeholder acceptable until Unsplash fetch
- [x] [Review][Defer] Filter chips above list deferred to Story 2.4
- [x] [Review][Defer] Full detail hero (Epic 3) will replace placeholder stub; transition names already aligned
