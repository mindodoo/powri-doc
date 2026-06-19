# Story 6.5: Trail Map Section — Button Only, No Inline Links in Copy

Status: done

baseline_commit: 59efbc4

## Story

As a **detail page reader**,  
I want the trail map section to use the CTA button only,  
So that I am not confused by duplicate link mentions in the descriptive text.

## Acceptance Criteria

1. **Given** a resort detail trail map section  
   **When** rendered  
   **Then** the **"View trail map"** button is the **only** clickable outbound link in that section

2. **And** `trail_map_note` prose contains **no raw URLs or domain names**

3. **And** all 20 published resorts audited and updated where needed

4. **And** `trail_map_opened` + `external_link_clicked` fire **only** on button click (unchanged)

5. **And** section shows **heading + button only** — `trail_map_note` is not rendered (editorial metadata only)

## Tasks / Subtasks

- [x] **Editorial pass** — rewrite 9 notes with embedded URLs/domains
- [x] **UI** — remove `trail_map_note` paragraph from `TrailMapSection` (button-only display)
- [x] **Launch gate check** — reject `trail_map_note` containing URL/domain patterns
- [x] **Template guidance** — `_TEMPLATE.md` comment updated
- [x] **Validate** — `test:launch`, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Resorts updated (URLs/domains removed)

Niseko United, Hakuba Valley, Furano, Shiga Kogen, Myoko Kogen, Tsugaike Kogen, Sahoro, Naeba, GALA Yuzawa

### Completion Notes

- `TrailMapSection` renders **heading + button only** — no note paragraph (avoids duplicate “trail map” copy beside the CTA)
- `trail_map_note` kept in resort markdown for editorial/launch-gate checks, not displayed
- Remaining 11 resorts already used descriptive copy without domains

### File List

- `content/resorts/*.md` (9 updated, 11 unchanged)
- `content/resorts/_TEMPLATE.md`
- `web/src/components/resort/TrailMapSection.tsx`
- `web/src/components/resort/ResortDetailContent.tsx`
- `web/scripts/verify-launch-gates.ts`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

Content + gate enforcement. No UI regression; analytics unchanged on button only.

### Review Findings

- [x] [Review][Pass] Grep: zero URL/domain patterns in `trail_map_note` across 20 resorts
- [x] [Review][Pass] `verify-launch-gates.ts` enforces Story 6.5 rule
- [x] [Review][Pass] `TrailMapSection` — single `<a>` on button
- [x] [Review][Pass] `test:launch`, lint, build green
