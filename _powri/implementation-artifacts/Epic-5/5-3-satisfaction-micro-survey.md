# Story 5.3: Satisfaction Micro-Survey

Status: done

baseline_commit: 48d7544

## Story

As a **product owner**,
I want lightweight satisfaction signal after meaningful browsing,
So that we can track the ≥4.2/5 confidence goal.

## Acceptance Criteria

1. **Given** a session where the user viewed a 2nd distinct resort detail page  
   **When** the second detail view completes  
   **Then** a once-per-session prompt asks "How helpful is this for planning your trip?" (1–5 stars) (§2, NFR-13)

2. **Given** the user submits a rating  
   **When** a star is selected  
   **Then** `satisfaction_prompt_submitted` fires with `score` and `resort_views_before_prompt`

3. **Given** the user dismisses the prompt  
   **When** dismissed  
   **Then** it does not re-prompt in the same session

4. **Given** the prompt is visible  
   **When** the user navigates  
   **Then** core navigation is not blocked (non-modal banner above bottom nav)

## Tasks / Subtasks

- [x] **Session tracking** (AC: 1, 3)
  - [x] `recordDistinctResortView(slug)` in sessionStorage
  - [x] Shown + handled flags — once per session; dismiss/submit sets handled

- [x] **SatisfactionPrompt UI** (AC: 1, 4)
  - [x] Fixed banner above bottom nav; 1–5 star tap targets
  - [x] "Not now" dismiss

- [x] **Wire detail page + analytics** (AC: 2)
  - [x] Extended `ResortDetailViewTracker`; fires event on star tap

- [x] **i18n + validate**
  - [x] `satisfaction.*` strings
  - [x] `npm run build` + `npm run lint`

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Prompt appears after 2nd **distinct** resort detail view in session
- sessionStorage tracks viewed slugs, shown, and handled state
- Star tap submits immediately; dismiss marks handled without analytics event
- Non-modal `role="region"` banner above bottom nav (same offset pattern as consent banner)

### File List

- `web/messages/en.json` (modified — `satisfaction.*`)
- `web/src/lib/satisfaction/session.ts` (new)
- `web/src/components/satisfaction/SatisfactionPrompt.tsx` (new)
- `web/src/components/resort/ResortDetailViewTracker.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Distinct resort counting, once-per-session prompt on 2nd view, analytics on submit, dismiss suppresses re-prompt, non-blocking banner UI.

### Review Findings

- [x] [Review][Pass] `resort_views_before_prompt` equals distinct count at prompt time (≥2)
- [x] [Review][Pass] Re-visiting same slug does not increment distinct count
- [x] [Review][Pass] Navigating away without dismiss sets `shown` — no repeat on 3rd resort
- [x] [Review][Defer] No event when consent denied — consistent with existing `track()` behaviour
- [x] [Review][Note] All five stars use filled icon — score communicated via `aria-label` only
