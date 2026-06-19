# Story 2.2.1: Compact Home Hero Height (30vh)

Status: done

baseline_commit: NO_VCS

## Story

As a **user on Home**,
I want the hero image to occupy roughly the top 30% of the viewport,
So that resort discovery content is visible without unnecessary scrolling.

## Acceptance Criteria

1. **Given** Home on a 375–430px viewport  
   **When** the page loads  
   **Then** the hero carousel height is **40vh** (not ~65vh)

2. **Given** the reduced hero  
   **When** tagline and Unsplash attribution render  
   **Then** copy remains legible (gradient overlay, no clipped text)

3. **Given** the fixed HomeTopBar scroll transition  
   **When** the user scrolls  
   **Then** the transparent→white transition still triggers at the correct hero boundary (30vh)

4. **Given** the change  
   **When** build runs  
   **Then** no layout regression on discovery section below hero

## Tasks / Subtasks

- [x] **Shared hero height constant** (AC: 1, 3)
  - [x] Add `--hero-height-home: 30vh` to `tokens.css`
  - [x] Export `HOME_HERO_VH_RATIO = 0.3` + `getHomeHeroHeightPx()` for scroll math

- [x] **HeroCarousel** (AC: 1, 2)
  - [x] Replace `h-[65vh] min-h-[320px]` with `var(--hero-height-home)` + `min-h: 11rem`
  - [x] Tagline → section-heading scale; tighter overlay padding

- [x] **HomeTopBar** (AC: 3)
  - [x] Scroll threshold uses `getHomeHeroHeightPx()`; resize listener added

- [x] **Validate** (AC: 4)
  - [x] `npm run build` + `npm run lint` pass

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Hero reduced from 65vh (~60%+ viewport) to 30vh per product feedback
- Single CSS token `--hero-height-home` keeps carousel and scroll math in sync
- Tagline scaled down to section-heading (26px) for compact overlay legibility

### File List

- `web/src/styles/tokens.css` (modified — `--hero-height-home`)
- `web/src/lib/home/heroLayout.ts` (new)
- `web/src/components/home/HeroCarousel.tsx` (modified)
- `web/src/components/home/HomeTopBar.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Hero uses 30vh token; tagline/attribution fit compact overlay; top bar scroll threshold aligned via shared helper.

### Review Findings

- [x] [Review][Defer] PRD/DESIGN.md still reference 65vh — update planning docs when UX spec is formally revised
- [x] [Review][Defer] `min-h: 11rem` floor on very short viewports — acceptable; revisit if attribution clips on landscape mobile
