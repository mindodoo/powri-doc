# Story UX-11: Responsive resort list grid

Status: ready-for-dev

Version when done: **2.12.2.11**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a tablet or desktop user browsing resorts,  
I want a multi-column card grid,  
so that Home, Explore, and Saved are not a single skinny column.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-11).

## Acceptance Criteria

1. **Home, Explore, Saved** lists use a **responsive grid from `md` up** (example: 3 columns tablet, 4 columns large desktop). Exact breakpoints: Tailwind `md` / `lg` / `xl` — pick values that don’t crush `ResortCard`.
2. **Mobile (`< md`):** **single column** — same as today (`flex-col` / `grid-cols-1`).
3. **Card internals unchanged** (image, title, heart, tags) except what the grid width requires (image can fill cell).
4. Home “Show all resorts” CTA and Explore “Show more” stay **full width below the grid**, not inside a cell.
5. Quiz results / other `ResortList` contexts: apply the same grid if they use `ResortList`; do not special-case unless cards overflow.

## Testing & Definition of Done

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A (layout only)
- [ ] **Analytics:** `resort_card_tapped` position still index in visible list
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual 375 / 768 / 1280. `e2e/smoke.spec.ts` card click still works
- [ ] **Manual QA:** PO — column counts

## Tasks / Subtasks

- [ ] `ResortList` `<ul>` → `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4` (or equivalent) (AC: 1–2)
- [ ] Verify Home / Explore / Saved all use `ResortList` (AC: 1)
- [ ] CTA full width (AC: 4)

## Dev Notes

**Today:** `ResortList` `ul` is `flex flex-col gap-card-gap`. Used from home, explore, saved (`listContext`). Cards are full-width magazine rows — on a 4-col grid they become tiles; **do not redesign** the card, accept narrower text.

**Do not:** Horizontal scroll. Masonry. Change `HOME_INITIAL_VISIBLE` (5) semantics.

### Files

| Path | Role |
|------|------|
| `web/src/components/resort/ResortList.tsx` | UPDATE ul classes |
| `web/src/components/resort/ResortCard.tsx` | Only if width breaks |
| Saved/Home/Explore pages | Confirm they use `ResortList` |

### References

- PRD UX-11

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
