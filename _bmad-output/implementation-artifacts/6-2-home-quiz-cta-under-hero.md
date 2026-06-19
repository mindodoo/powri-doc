# Story 6.2: Home Quiz CTA Under Hero

Status: done

baseline_commit: 9718039

## Story

As an **undecided planner**,  
I want a clear quiz entry point on Home below the hero,  
So that I can start "Find my resort" without opening Explore first.

## Acceptance Criteria

1. **Given** the Home page below `HeroCarousel` and above resort discovery  
   **When** the section renders  
   **Then** heading copy **"Find your right ski resort"** displays (`home.quizCtaHeading`)

2. **And** primary button **"Find my resort"** links to **`/quiz?from=home`**

3. **And** tapping the button does **not** fire `quiz_started` (only quiz intro start does)

4. **And** **`quiz_cta_tapped`** fires with `entry_point: home_hero`, `surface: home`

5. **And** full-width button, min 52px tap target, Theme A spacing

6. **And** Explore tab quiz CTA unchanged (separate placement)

## Tasks / Subtasks

- [x] **`HomeQuizCta` component** (AC: 1–5)
- [x] **Wire on Home page** between hero and discovery (AC: 1)
- [x] **i18n** — `home.quizCtaHeading`, `home.quizCtaButton`
- [x] **Analytics** — `quiz_cta_tapped` in registry + tracking-plan §2
- [x] **Validate** — `test:analytics`, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Reuses accent button styling from Explore `FindMyResortCta` (Sparkles icon + `bg-accent`)
- Quiz page already accepts `from=home` for `quiz_started` entry_point

### File List

- `web/src/components/home/HomeQuizCta.tsx` (new)
- `web/src/app/[locale]/page.tsx`
- `web/messages/en.json`
- `web/src/lib/analytics/phase1Events.ts`
- `docs/tracking-plan.md`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

All six ACs satisfied. CTA placement is Home-only; analytics correctly separates tap intent (`quiz_cta_tapped`) from quiz start (`quiz_started`).

### Review Findings

- [x] [Review][Pass] Section between hero and discovery with semantic heading + labelled section
- [x] [Review][Pass] Links to `/quiz?from=home`; entry_point wired in quiz page
- [x] [Review][Pass] `quiz_cta_tapped` in registry + tracking-plan
- [x] [Review][Pass] `test:analytics`, lint, build green
- [x] [Review][Note] Explore CTA unchanged at `/quiz?from=explore`
