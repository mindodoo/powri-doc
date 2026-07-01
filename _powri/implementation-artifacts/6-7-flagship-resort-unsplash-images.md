# Story 6.7: Flagship Resort Unsplash Images (Phase A)

Status: done

baseline_commit: 04b3226

## Story

As a **visitor**,  
I want real resort photography on flagship destinations,  
So that discovery and detail feel credible before all 20 are illustrated.

## Acceptance Criteria

**Phase A — minimum (launch polish):**

1. **Given** Niseko United, Hakuba Valley, Nozawa Onsen  
   **When** content is updated  
   **Then** each has ≥1 **`images.hero`** entry with full Unsplash attribution metadata

2. **And** `images.card` populated (hero[0] mirror)

3. **And** detail hero + list cards show real images with **`UnsplashAttribution`** visible

4. **And** `npm run test:launch` — warnings reduced for completed slugs (17 resorts still placeholder)

## Tasks / Subtasks

- [x] **Content** — hero + card metadata for niseko-united, hakuba-valley, nozawa-onsen
- [x] **CDN map** — `UNSPLASH_CDN_BY_PHOTO_ID` entries in `resortListItem.ts`
- [x] **UI** — `UnsplashAttribution` on `ResortCard` + `QuizTopMatchCard` when image present
- [x] **Launch gate** — add `ResortCard.tsx` to NFR-7 UI check
- [x] **Report** — `content/reports/unsplash-flagship-selections.json`
- [x] **Validate** — `test:launch`, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Photo selections

| Slug | photo_id | Photographer | Notes |
|------|----------|--------------|-------|
| niseko-united | RivtzPBt58o | Marek Okon | Niseko, Hokkaido snowfall |
| hakuba-valley | fhbn0qpygwE | Colton Jones | Hakuba mountain range |
| nozawa-onsen | z6aCyLMtTuY | Nopparuj Lamaikul | Representative onsen-town winter street (no exact Nozawa geo on Unsplash) |

### Completion Notes

- Detail hero attribution unchanged (`ResortDetailHero` + `getResortCardImage` fallback hero[0])
- Unsplash download endpoint not triggered — pending `UNSPLASH_ACCESS_KEY` / app approval (§17.2)
- Phase B (remaining 17 resorts) deferred to 6.7b

### File List

- `content/resorts/niseko-united.md`
- `content/resorts/hakuba-valley.md`
- `content/resorts/nozawa-onsen.md`
- `content/reports/unsplash-flagship-selections.json`
- `web/src/lib/content/resortListItem.ts`
- `web/src/components/resort/ResortCard.tsx`
- `web/src/components/quiz/QuizTopMatchCard.tsx`
- `web/scripts/verify-launch-gates.ts`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve (Phase A)

### Review Findings

- [x] [Review][Pass] Full attribution metadata on all 3 flagship slugs
- [x] [Review][Pass] CDN URLs resolve for hero/card images
- [x] [Review][Pass] `UnsplashAttribution` on detail hero + list cards
- [x] [Review][Pass] `test:launch` — 3 slugs no longer warn on missing images
- [x] [Review][Defer] Download endpoint trigger when API key available
