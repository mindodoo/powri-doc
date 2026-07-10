# Story 3.4: Getting There — Transport Options

Status: done

baseline_commit: 48d7544

## Story

As a **user planning logistics**,
I want clear transport options with time, cost, and booking links,
So that I know how to reach the resort from major gateways.

## Acceptance Criteria

1. **Given** a resort with ≥2 transport options  
   **When** I scroll to Getting There on the detail page  
   **Then** each option renders as `TransportRow` (icon, method, duration, cost, note) — **no booking chip in Phase 1** (FR-3, UX-DR10)

2. **Phase 1 scope:** Transport is informational only. Booking/monetisation link-outs are **explicitly out of scope** until a later phase (PRD §11.1 — no affiliate, booking widgets, or ads in Phase 1).

3. **Given** Getting There is visible  
   **When** the section enters view  
   **Then** `getting_there_viewed` fires once with `resort_slug`

4. **Given** fewer than 2 transport options  
   **When** the detail page loads  
   **Then** Getting There section is omitted

## Tasks / Subtasks

- [x] **TransportRow + GettingThereSection** (AC: 1)
  - [x] Mode-based icons; duration, cost, note — no booking chip (Phase 1)

- [x] **Analytics** (AC: 3)
  - [x] IntersectionObserver for `getting_there_viewed`

- [x] **Phase 1 correction (2026-06-13)**
  - [x] Removed Book chip + `booking_url` schema — monetisation/booking link-outs deferred to post-Phase 1

- [x] **Wire + i18n + validate**
  - [x] Section after trail map on detail page
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- Booking/monetisation chips **removed** — Phase 1 transport rows are informational (icon, mode, duration, cost, note) per PRD §11.1
- Gateway airport line + recommended transport callout above transport rows
- Gateway city cards + practical tips deferred to Story 3.5

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Getting There section with gateway summary, recommended transport highlight, and transport rows
- Each row: icon, mode, duration · cost, note (no booking actions in Phase 1)
- `getting_there_viewed` on section intersection

### File List

- `web/messages/en.json` (modified)
- `web/src/lib/resort/detailSections.ts` (modified — `gettingThere`)
- `web/src/components/resort/TransportRow.tsx` (new — no booking chip)
- `web/src/components/resort/GettingThereSection.tsx` (new)
- `web/src/components/resort/ResortDetailContent.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All ACs satisfied for Phase 1 scope. Getting There renders transport rows without booking/monetisation actions.

### Review Findings

- [x] [Review][Pass] Book chip removed — aligns with PRD §11.1 (no monetisation Phase 1)
- [x] [Review][Defer] Gateway city experience cards + practical tips — Story 3.5
- [x] [Review][Defer] Booking link-outs — post-Phase 1 monetisation phase only
- [x] [Review][Pass] All 20 published resorts have ≥2 `transport_options`
