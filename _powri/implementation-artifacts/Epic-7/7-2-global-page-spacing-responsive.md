# Story 7.2: Global Page Spacing — Top & Responsive Horizontal

Status: done

baseline_commit: b833449

## Story

As a **visitor on any device**,  
I want page content to sit closer to the top and use screen width sensibly on tablet/desktop,  
So that I see more content without odd empty margins.

## Acceptance Criteria

1. `--space-section-gap` reduced to **2rem (32px)**; `--space-section-gap-compact` added
2. Responsive `--content-max-width` (32rem → 42rem md → 48rem lg) via `max-w-content`
3. `--space-screen-x` **24px at md+**
4. Top spacing reduced on Home, Explore, Info, Quiz intro/results, Legal, Resort detail
5. Explore: no stacked top-bar offset **and** `pt-section-gap` on header
6. `lib/layout/pageInsets.ts` documents top-bar offset for 7.3/7.4

## Tasks / Subtasks

- [x] Tokens + `globals.css` theme
- [x] `pageInsets.ts` + `PageContent.tsx`
- [x] Migrate listed pages
- [x] Validate lint + build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- `--space-section-gap` 48px → 32px globally; `--space-section-gap-compact` (24px) for Home quiz CTA + Info footer
- Responsive `--content-max-width` + `--space-screen-x` at `md`/`lg` in `tokens.css`; Tailwind `max-w-content`
- `PageContent` wrapper replaces repeated `px-screen-x` + `max-w-lg` patterns
- Explore: `TOP_BAR_OFFSET_CLASS` + `BELOW_TOP_BAR_CLASS` (`pt-4`) only — removed header `pt-section-gap` double stack (~80px saved)
- Bottom nav / consent / satisfaction align to `max-w-content`

### File List

- `web/src/styles/tokens.css`
- `web/src/app/globals.css`
- `web/src/lib/layout/pageInsets.ts` (new)
- `web/src/components/layout/PageContent.tsx` (new)
- `web/src/app/[locale]/page.tsx`
- `web/src/app/[locale]/explore/page.tsx`
- `web/src/app/[locale]/info/page.tsx`
- `web/src/components/legal/LegalPageLayout.tsx`
- `web/src/components/resort/ResortDetailContent.tsx`
- `web/src/components/home/HomeQuizCta.tsx`
- `web/src/components/quiz/QuizFlow.tsx`
- `web/src/components/quiz/QuizResultsReveal.tsx`
- `web/src/components/layout/BottomNav.tsx`
- `web/src/components/analytics/ConsentBanner.tsx`
- `web/src/components/satisfaction/SatisfactionPrompt.tsx`

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve

### Summary

Global spacing tokens tightened; content column widens on tablet/desktop. Explore double-padding fixed. Shared `PageContent` + `pageInsets` give 7.3/7.4 a single offset source.

### AC traceability

| AC | Status | Notes |
|----|--------|-------|
| 1 | Pass | `section-gap` 32px; `section-gap-compact` 24px |
| 2 | Pass | `max-w-content` 512 / 672 / 768px |
| 3 | Pass | `screen-x` 20px → 24px at `md` |
| 4 | Pass | All listed pages use `PageContent` or compact tokens |
| 5 | Pass | Explore: offset + `pt-4` only |
| 6 | Pass | `pageInsets.ts` exports `TOP_BAR_OFFSET_CLASS`, `BELOW_TOP_BAR_CLASS` |

### Review Findings

- [x] [Review][Pass] Token media queries in `tokens.css` — no duplicate gutters
- [x] [Review][Pass] `PageContent` `padY={false}` pattern for fixed-top-bar pages
- [x] [Review][Pass] Quiz 7.1b results `pt-4` preserved — not regressed by global `section-gap` change
- [x] [Review][Pass] `npm run lint`, `npm run build` green
- [x] [Review][Note] Quiz questions phase still uses full-width card padding — intentional; centered layout unchanged from 7.1
