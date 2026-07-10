# Story 6.12: Detail Hero Quick Verdict — Smaller Type

Status: done

baseline_commit: 9518613

## Story

As a **detail page reader**,  
I want the quick verdict pull quote in a smaller type size,  
So that long resort summaries do not push the resort name into the back-button area on mobile.

## Acceptance Criteria

1. **Given** a resort detail hero with quick verdict  
   **When** rendered at 375px  
   **Then** verdict uses `--font-size-quick-verdict` (13px), not 24px display-italic

2. **And** verdict clamped to 3 lines max

3. **And** Joetsu Kokusai `name_en` is `Joetsu Kokusai` with PRD/Nendai notes removed

## Tasks / Subtasks

- [x] **QuickVerdict** — new token + `line-clamp-3`
- [x] **Joetsu Kokusai** — rename + scrub Nendai references in frontmatter and body
- [x] **Validate** — lint, build

## Dev Agent Record

### Agent Model Used

Composer

### File List

- `web/src/styles/tokens.css`
- `web/src/components/resort/QuickVerdict.tsx`
- `content/resorts/joetsu-kokusai.md`

## Senior Developer Review (AI)

**Review date:** 2026-06-16  
**Outcome:** Approve

### Review Findings

- [x] [Review][Pass] QuickVerdict uses 13px token (down from 24px)
- [x] [Review][Pass] line-clamp-3 limits hero stack height
- [x] [Review][Pass] Joetsu name + copy cleaned
