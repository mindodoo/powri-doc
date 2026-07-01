# Story 6.6: Highlight / Lowlight Row — Column Header Icons

Status: done

baseline_commit: 25ef7e7

## Story

As a **detail page reader**,  
I want visual cues on the highlights and watch-outs column headers,  
So that the honesty grid scans faster on mobile without cluttering each bullet.

## Acceptance Criteria

1. **Given** `HighlightLowlightRow` with non-empty highlights or lowlights  
   **When** rendered  
   **Then** each column header includes a **16px Lucide icon** before the label text

2. **And** highlights header uses `CircleCheck` in `{colors.accent}`

3. **And** lowlights header uses `AlertTriangle` in `{colors.accent-warm}`

4. **And** bullet rows remain **text-only** (no per-item icons)

5. **And** no layout regression at 375px (50/50 columns preserved)

## Tasks / Subtasks

- [x] **UI** — add `CircleCheck` / `AlertTriangle` to column headers in `HighlightLowlightRow`
- [x] **Revert per-item icons** — bullet rows text-only (PO feedback)
- [x] **Validate** — lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Icons on column headers only (`flex items-center gap-1.5` on `<h3>`)
- Bullet rows restored to text-only tinted pills
- Column grid (`grid-cols-2 gap-3`) unchanged — 50/50 layout preserved at 375px
- No content schema changes

### File List

- `web/src/components/resort/HighlightLowlightRow.tsx`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve (revised per PO feedback)

### Summary

Header-only icons reduce visual noise while keeping column scan cues. Per-item icons removed after review.

### Review Findings

- [x] [Review][Pass] Highlights header: `CircleCheck` 16px, `text-accent`
- [x] [Review][Pass] Lowlights header: `AlertTriangle` 16px, `text-accent-warm`
- [x] [Review][Pass] Bullet rows text-only
- [x] [Review][Pass] `grid-cols-2` layout unchanged
- [x] [Review][Pass] lint, build green
