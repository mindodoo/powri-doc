# Story 3.5: Gateway City Experiences & Practical Tips

Status: done

baseline_commit: 48d7544

## Story

As a **user flying into Japan**,
I want gateway city highlights and collapsible practical tips,
So that I can plan arrival days and avoid cultural pitfalls.

## Acceptance Criteria

1. **Given** ≥3 `nearby_daytrips` entries  
   **When** I scroll Getting There  
   **Then** experience cards show name + 1-line description; "See more" if >3 (FR-3)

2. **Given** practical tips  
   **When** rendered  
   **Then** collapsible FAQ rows (cash/card, onsen, JR Pass, IC Card, language) — mobile-friendly (FR-3)

3. **UnsplashAttribution** on experience cards when image metadata exists (NFR-7)

## Tasks / Subtasks

- [x] **Gateway experiences** (AC: 1, 3)
  - [x] Map `nearby_daytrips` → cards; parse `(detail)` suffix as description
  - [x] `GatewayExperienceCard`, `GatewayExperiencesBlock` with See more/less
  - [x] Attribution wired when image metadata present (placeholder icon until images)

- [x] **Practical tips** (AC: 2)
  - [x] `PracticalTipsBlock` — native `<details>` FAQ rows
  - [x] Shared Phase 1 copy in i18n (Japan-wide tips)

- [x] **Wire + validate**
  - [x] Below transport rows in `GettingThereSection`
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- Experience data source: `nearby_daytrips` frontmatter (CONTENT_MODEL §4.8) — no separate gateway_experiences schema yet
- Resorts with <3 daytrips (e.g. GALA Yuzawa) omit gateway cards; practical tips still show
- Practical tips are app-wide Phase 1 copy, not per-resort markdown

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Gateway cards from `nearby_daytrips` with parenthetical descriptions parsed out
- See more expands beyond 3 cards (e.g. Niseko has 4)
- Five collapsible practical tips below gateway/transport block

### File List

- `web/messages/en.json` (modified — gateway + practicalTips)
- `web/src/lib/resort/gatewayExperiences.ts` (new)
- `web/src/lib/resort/detailSections.ts` (modified)
- `web/src/components/resort/GatewayExperienceCard.tsx` (new)
- `web/src/components/resort/GatewayExperiencesBlock.tsx` (new)
- `web/src/components/resort/PracticalTipsBlock.tsx` (new)
- `web/src/components/resort/GettingThereSection.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All ACs satisfied. Gateway experience cards from daytrips with expand/collapse; practical tips as mobile-friendly FAQ; attribution ready when images added.

### Review Findings

- [x] [Review][Defer] Experience card images use placeholder icon — UnsplashAttribution shows when `imageAttribution` populated
- [x] [Review][Defer] GALA Yuzawa / Joetsu Kokusai have <3 daytrips — gateway cards hidden; tips still show
- [x] [Review][Note] Practical tips are shared i18n copy, not per-resort — acceptable for Phase 1
- [x] [Review][Pass] Native `<details>` avoids horizontal scroll on mobile
