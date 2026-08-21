---
baseline_commit: c6c5f1f5588007ba3bc68e484c844c3d2a2ce2ab
---

# Story UX-3: Around the resort after Trailmap

Status: done

Version when done: **2.12.2.3**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a skier reading resort detail,  
I want nearby places right after the trail map,  
so that the page flows mountain → map of the hill → what’s around.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-3).

## Acceptance Criteria

1. **Around the resort** (`ResortMapSection` — map + place list) renders **immediately after** `TrailMapSection`.
2. Intended order: season / highlights / terrain → **Trail map** → **Around the resort** → Getting there → Experience → practical tips (tips stay after experience unless already coupled to getting-there).
3. **Reorder only** — no new map features, pins, or Places API changes.
4. If trail map URL is missing, Around the resort still renders where trail map would have been (after terrain, before Getting there).
5. If no lat/lng (`hasMap` false), skip map section as today.

## Testing & Definition of Done

- [x] **Unit:** N/A (JSX order)
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A (do not regress `deriveAroundArea`)
- [x] **User-facing flow:** No e2e/unit selectors assumed Getting there → map order. Smoke `resort detail loads from direct URL` passed. Manual: Niseko detail scroll
- [x] **Manual QA:** PO — section sequence

## Tasks / Subtasks

- [x] Move `{hasMap ? <ResortMapSection /> : null}` in `ResortDetailContent.tsx` to immediately after `TrailMapSection` (AC: 1–5)
- [x] Grep tests/e2e for “Getting there” then map order
- [x] lint + build + relevant e2e

## Dev Notes

**Today** (`ResortDetailContent.tsx`): TrailMap → **ResortMapSection** → GettingThere → Experience → PracticalTips.

**Do not:** Change `GettingThereSection` internals, Epic 11 map behaviour, or Experience placement relative to map beyond this reorder (Experience stays after Getting there).

### Files

| Path | Role |
|------|------|
| `web/src/components/resort/ResortDetailContent.tsx` | UPDATE order |

### References

- PRD UX-3
- Epic 11 `ResortMapSection`

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (dev-story)

### Debug Log References

- Playwright browsers missing in sandbox `PLAYWRIGHT_BROWSERS_PATH`; installed chromium v1234 to user cache and re-ran with path unset.
- `npm run test:launch` fails on missing `NEXT_PUBLIC_CONTACT_EMAIL` (pre-existing env; not caused by this reorder).

### Completion Notes List

- Reordered `ResortMapSection` to immediately follow `TrailMapSection` in `ResortDetailContent.tsx`. `hasMap` skip unchanged. PracticalTips still after Experience, gated on `gettingThere`.
- No e2e/unit tests encoded Getting there → map DOM order; no selector updates.
- lint (0 errors, pre-existing warnings), `npm run build` pass; smoke resort-detail e2e pass.

### File List

- `web/src/components/resort/ResortDetailContent.tsx`
- `_powri/implementation-artifacts/UX/ux-3-around-resort-after-trailmap.md`
- `_powri/implementation-artifacts/sprint-status.yaml`

### Change Log

- 2026-08-21: UX-3 — Around the resort now renders after Trail map, before Getting there.
