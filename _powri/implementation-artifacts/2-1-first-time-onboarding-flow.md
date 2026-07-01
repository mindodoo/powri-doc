# Story 2.1: First-Time Onboarding Flow

Status: done

baseline_commit: NO_VCS

## Story

As a **first-time visitor**,
I want a brief swipeable introduction to the app,
So that I understand what the product offers before browsing resorts.

## Acceptance Criteria

1. **Given** onboarding is not complete (`localStorage` key unset)  
   **When** I open the app  
   **Then** I see ≤4 full-screen `OnboardingCard` slides with icon, title, and body — no photographs (FR-5, UX-DR11)

2. **Given** the onboarding overlay  
   **When** I interact  
   **Then** progress dots, skip (top-right), and "Get started" on the final card all work

3. **Given** I skip or complete onboarding  
   **When** the flow ends  
   **Then** `onboarding_complete` is set in `localStorage` and onboarding never shows again

4. **Given** the final card  
   **When** I tap "Find my resort"  
   **Then** onboarding completes and navigates to `/explore` (quiz CTA wired in Stories 2.6–2.7)

## Tasks / Subtasks

- [x] **Storage helper** (AC: 3)
  - [x] `web/src/lib/onboarding/storage.ts` — key `onboarding_complete` per architecture.md

- [x] **OnboardingCard component** (AC: 1)
  - [x] `web/src/components/onboarding/OnboardingCard.tsx` — icon 80×80, title, body, `--color-bg`, no photos

- [x] **OnboardingFlow** (AC: 1, 2, 4)
  - [x] `web/src/components/onboarding/OnboardingFlow.tsx` — 4 slides, swipe/tap advance, dots, skip, CTAs
  - [x] Final card: "Get started" + "Find my resort" → `/explore`

- [x] **OnboardingGate** (AC: 1, 3)
  - [x] `web/src/components/onboarding/OnboardingGate.tsx` — client overlay in AppShell via `useSyncExternalStore`
  - [x] Hydration-safe: no flash on repeat visits

- [x] **i18n** (AC: 1)
  - [x] Add `onboarding` keys to `messages/en.json`

- [x] **Validate** (AC: 1–4)
  - [x] `npm run build` + `npm run lint` pass

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- 4 slides with Lucide icons (Mountain, Compass, Map, MessageCircle)
- Touch swipe ±50px threshold + Next button; skip completes flow
- `useSyncExternalStore` pattern matches analytics consent (no setState-in-effect lint)
- Full-screen z-[100] overlay above nav and consent banner

### File List

- `web/messages/en.json` (modified)
- `web/src/lib/onboarding/storage.ts` (new)
- `web/src/components/onboarding/OnboardingCard.tsx` (new)
- `web/src/components/onboarding/OnboardingFlow.tsx` (new)
- `web/src/components/onboarding/OnboardingGate.tsx` (new)
- `web/src/components/layout/AppShell.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Four icon-only slides, swipe + Next navigation, skip/Get started/Find my resort CTAs, `onboarding_complete` persistence via `useSyncExternalStore` (matches consent pattern).

### Review Findings

- [x] [Review][Defer] `aria-labelledby="onboarding-slide-title"` set but `h1` lacks matching `id` — fix in polish pass
- [x] [Review][Defer] No keyboard slide navigation — acceptable for V1; revisit for NFR-10
- [x] [Review][Defer] Onboarding shows on all routes via AppShell — correct for first app open; no route-scoping needed
