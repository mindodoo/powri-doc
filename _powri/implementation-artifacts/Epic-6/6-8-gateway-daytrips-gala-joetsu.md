# Story 6.8: Gateway Daytrips — GALA Yuzawa & Joetsu Kokusai

Status: done

baseline_commit: 51f7a3f

## Story

As a **reader on thinner Getting There pages**,  
I want gateway experience cards where other resorts have them,  
So that the detail page feels consistent.

## Acceptance Criteria

1. **Given** `gala-yuzawa.md` and `joetsu-kokusai.md`  
   **When** editorial pass completes  
   **Then** each has **≥3** entries in **`nearby_daytrips`**

2. **And** `GatewayExperiencesBlock` renders cards on both detail pages (≥3 threshold)

3. **And** format matches other resorts (`Name (detail)` where helpful)

## Tasks / Subtasks

- [x] **GALA Yuzawa** — expand to 4 Yuzawa-cluster daytrips
- [x] **Joetsu Kokusai** — expand to 4 Niigata/Yuzawa-area daytrips
- [x] **Validate** — gateway experience count ≥3; lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Editorial notes

- GALA: Yuzawa town, linked Ishiuchi/Yuzawa Kogen, Echigo-Yuzawa station hub
- Joetsu Kokusai: Echigo-Yuzawa, GALA, Naeba/Kagura cluster, Minami-Uonuma rice/sake

### File List

- `content/resorts/gala-yuzawa.md`
- `content/resorts/joetsu-kokusai.md`

## Senior Developer Review (AI)

**Review date:** 2026-06-16  
**Outcome:** Approve

### Review Findings

- [x] [Review][Pass] GALA: 4 `nearby_daytrips` entries
- [x] [Review][Pass] Joetsu Kokusai: 4 entries
- [x] [Review][Pass] `toGatewayExperiences()` ≥3 for both slugs
- [x] [Review][Pass] lint, build green
