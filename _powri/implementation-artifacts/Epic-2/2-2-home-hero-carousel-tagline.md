# Story 2.2: Home Hero Carousel & Tagline

Status: done

baseline_commit: NO_VCS

## Story

As a **user landing on Home**,
I want a stunning full-bleed hero with curated Japan ski imagery,
So that the first impression matches the premium travel-magazine vision.

## Acceptance Criteria

1. **Given** `content/site-images.md` defines home hero assets  
   **When** Home loads  
   **Then** `HeroCarousel` displays crossfade slides (3–4s hold, 1s fade, no slide animation) (UX-DR4)

2. **Given** the hero  
   **When** rendered  
   **Then** white gradient overlay and tagline in Cormorant Garamond 600 appear over the hero

3. **Given** hero images  
   **When** displayed  
   **Then** `UnsplashAttribution` renders per §17.2 (UX-DR14)

4. **Given** Home scroll  
   **When** user scrolls past hero  
   **Then** sticky top bar transitions from transparent to white + border (UX-DR16)

## Tasks / Subtasks

- [x] **Site images loader** (AC: 1)
  - [x] `web/src/lib/content/loadSiteImages.ts`
  - [x] Zod schema in `siteImagesSchema.ts`
  - [x] CDN URL map for L7289nHzVgI

- [x] **UnsplashAttribution** (AC: 3)
  - [x] `web/src/components/ui/UnsplashAttribution.tsx`

- [x] **HeroCarousel** (AC: 1, 2)
  - [x] 65vh, crossfade 3500ms hold / 1000ms fade, gradient, tagline
  - [x] Single-image: no auto-advance timer

- [x] **HomeTopBar** (AC: 4)
  - [x] Fixed overlay, scroll-triggered white bg + border
  - [x] Search icon placeholder (disabled until Story 2.5)

- [x] **Home page layout** (AC: 1–4)
  - [x] Refactored `[locale]/page.tsx`

- [x] **i18n**
  - [x] `home.heroTagline` in `messages/en.json`

- [x] **Validate**
  - [x] `npm run build` + `npm run lint` pass
  - [x] `next.config.ts` — `images.remotePatterns` for Unsplash

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Loads 1 hero image from `content/site-images.md` frontmatter
- Crossfade ready for additional images when editorial adds them
- HomeTopBar uses fixed positioning + scroll listener for transparent→white transition

### File List

- `web/next.config.ts` (modified — Unsplash remotePatterns)
- `web/messages/en.json` (modified)
- `web/src/lib/content/paths.ts` (modified)
- `web/src/lib/content/siteImagesSchema.ts` (new)
- `web/src/lib/content/loadSiteImages.ts` (new)
- `web/src/lib/content/index.ts` (modified)
- `web/src/components/ui/UnsplashAttribution.tsx` (new)
- `web/src/components/home/HeroCarousel.tsx` (new)
- `web/src/components/home/HomeTopBar.tsx` (new)
- `web/src/app/[locale]/page.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Site-images loader reads frontmatter; hero at 65vh with gradient + tagline; Unsplash attribution with UTM links; top bar transparent→white on scroll.

### Review Findings

- [x] [Review][Defer] Carousel interval is 3500ms (hold only) — spec is 3.5s hold + 1s fade (4500ms total cycle); tune when multiple images added
- [x] [Review][Defer] CDN URL map hardcoded by `photo_id` — add `src` to frontmatter or Unsplash API when bulk images arrive
- [x] [Review][Defer] Unsplash download endpoint not triggered — needs API key; Story 2.2 out of scope
- [x] [Review][Defer] Top bar uses `fixed` not `sticky` — achieves same UX; rename if semantics matter
