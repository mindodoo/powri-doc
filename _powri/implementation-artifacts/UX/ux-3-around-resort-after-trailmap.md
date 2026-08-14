# Story UX-3: Around the resort after Trailmap

Status: ready-for-dev

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

- [ ] **Unit:** N/A (JSX order)
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** N/A
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A (do not regress `deriveAroundArea`)
- [ ] **User-facing flow:** Existing map/e2e if they assume DOM order — update selectors. Manual: Niseko detail scroll
- [ ] **Manual QA:** PO — section sequence

## Tasks / Subtasks

- [ ] Move `{hasMap ? <ResortMapSection /> : null}` in `ResortDetailContent.tsx` to immediately after `TrailMapSection` (AC: 1–5)
- [ ] Grep tests/e2e for “Getting there” then map order
- [ ] lint + build + relevant e2e

## Dev Notes

**Today** (`ResortDetailContent.tsx`): TrailMap → GettingThere → **ResortMapSection** → Experience → PracticalTips.

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

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
