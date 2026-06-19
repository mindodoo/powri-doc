# Story 3.1: Resort Detail Layout, Hero & Quick Verdict

Status: done

baseline_commit: 48d7544

## Story

As a **user who tapped a resort card**,
I want a seamless hero transition and an immediate editorial verdict,
So that I feel confident I'm viewing the right destination.

## Acceptance Criteria

1. **Given** I navigate to `/resorts/[slug]`  
   **When** the detail page loads  
   **Then** full-bleed hero (~55vh) uses the **same image as the list card** with shared-element transition (`viewTransitionName: resort-hero-{slug}`) (FR-2, §12.5 Screen 2)

2. **Given** the detail hero  
   **When** rendered  
   **Then** resort name (Cormorant Garamond 600), region pill, and back control overlay the hero

3. **Given** resort content includes `quick_verdict`  
   **When** the hero renders  
   **Then** `QuickVerdict` italic pull quote sits on lower hero with dark gradient backing (UX-DR7)

4. **Given** hero image has Unsplash metadata  
   **When** the detail page loads  
   **Then** `UnsplashAttribution` is visible on the hero (NFR-7, UX-DR14)

5. **Given** the detail page opens  
   **When** mounted  
   **Then** `resort_detail_viewed` fires once with `resort_slug` and `source` (`card` | `search` | `deep_link` via `?source=`)

## Tasks / Subtasks

- [x] **Detail hero layout** (AC: 1, 2)
  - [x] `--hero-height-detail: 55vh` token; `ResortDetailHero` component
  - [x] Same image source as list card (`getResortCardImage`); matching transition name
  - [x] Back button (client), region pill, name overlay on gradient

- [x] **QuickVerdict** (AC: 3)
  - [x] `QuickVerdict.tsx` — italic serif on hero lower third
  - [x] Stronger detail-hero gradient for text legibility

- [x] **Attribution** (AC: 4)
  - [x] Wire `UnsplashAttribution` when card image metadata present

- [x] **Analytics + source** (AC: 5)
  - [x] `ResortDetailViewTracker` client component
  - [x] `?source=search` on search result links

- [x] **Page refactor + i18n**
  - [x] Replace placeholder detail layout in `resorts/[slug]/page.tsx`
  - [x] Remove placeholder body copy; 3.2 adds scroll sections below hero
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- **UX reference:** `mockups/key-resort-detail.html` — back, region pill, name, verdict on hero; white content below starts at 3.2
- **Image parity:** Use `getResortCardImage()` + `getResortImageSrc()` — NOT `images.hero[]` (empty in content). Ensures card→detail transition matches.
- **Existing:** `viewTransitionName` already paired in `ResortCard` and detail stub (Story 2.3); `experimental.viewTransition` in `next.config.ts`
- **UnsplashAttribution:** Map `ResortImage` camelCase → component props; hide when no image metadata
- **Source param:** Valid values per `docs/tracking-plan.md`: `card`, `search`, `chat`, `deep_link`. Default `card` for in-app navigation without query param.
- **Out of scope (3.2+):** Season snapshot, highlights, terrain, trail map, Getting There, AI float

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story 3.1]
- [Source: `mockups/key-resort-detail.html`]
- [Source: `docs/tracking-plan.md` — `resort_detail_viewed`]
- [Source: `web/src/components/resort/ResortCard.tsx` — transition naming]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Replaced placeholder detail page with 55vh hero overlay layout matching mockup spine
- Quick verdict pulled from markdown body; displays on hero with decorative rule
- `resort_detail_viewed` fires once on mount; search navigations pass `?source=search`
- Scroll content (season, highlights, terrain) deferred to Story 3.2

### File List

- `web/messages/en.json` (modified)
- `web/src/styles/tokens.css` (modified — detail hero height + gradient)
- `web/src/lib/content/resortDetailHero.ts` (new)
- `web/src/lib/content/index.ts` (modified)
- `web/src/components/resort/QuickVerdict.tsx` (new)
- `web/src/components/resort/ResortDetailBack.tsx` (new)
- `web/src/components/resort/ResortDetailHero.tsx` (new)
- `web/src/components/resort/ResortDetailViewTracker.tsx` (new)
- `web/src/components/home/SearchResultRow.tsx` (modified — source=search)
- `web/src/app/[locale]/resorts/[slug]/page.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All five ACs satisfied. Detail hero uses 55vh, card-parity image + transition name, overlay name/region/back, QuickVerdict on gradient, attribution wired when image metadata exists, analytics on mount with source param support.

### Review Findings

- [x] [Review][Defer] No resort images in content yet — gradient placeholder + no attribution until Unsplash fetch (Epic 3 content/images)
- [x] [Review][Defer] `deep_link` source not auto-detected — only via explicit `?source=`; default `card` covers in-app navigation
- [x] [Review][Defer] Body sections below hero empty until Story 3.2 — expected
- [x] [Review][Pass] Shared-element transition names unchanged from Story 2.3
