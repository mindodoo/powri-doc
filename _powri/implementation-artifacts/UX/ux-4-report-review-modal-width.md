# Story UX-4: Report-review modal desktop width

Status: ready-for-dev

Version when done: **2.12.2.4**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a desktop user reporting a review,  
I want a properly wide modal,  
so that it reads as a dialog, not a small popover.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-4).

## Acceptance Criteria

1. **Mobile:** keep current treatment (`inset-x-4`, rounded sheet, existing type/spacing).
2. **Desktop (`md+`):** **wider** than today’s `max-w-[400px]` — use a modal width that fills a comfortable fraction of the content/dialog (e.g. `md:max-w-xl` or `md:w-[min(32rem,calc(100vw-4rem))]`). Centered (`left-1/2 -translate-x-1/2` already present).
3. Overlay, Escape, submit/duplicate/error states unchanged.
4. Do not change report API or reasons.

## Testing & Definition of Done

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** N/A (`content_reported` already wired)
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual desktop + mobile; `e2e/review-text-crud.spec.ts` only if it screenshots the dialog
- [ ] **Manual QA:** PO — desktop width

## Tasks / Subtasks

- [ ] Widen `ReportReviewDialog` panel at `md:` (AC: 1–2)
- [ ] Visual check mobile unchanged (AC: 1)

## Dev Notes

**Today:** `web/src/components/experience/ReportReviewDialog.tsx` dialog class includes `max-w-[400px]` + `md:inset-x-auto md:left-1/2 md:-translate-x-1/2`. Backdrop `z-[62]`.

**Reuse:** Existing dialog; do **not** pull in a new modal library. Sign-in desktop modal (`9-14`) is a separate component — match *feel* (proper width), don’t merge code unless trivial.

**Do not:** Full-screen desktop. Change z-index stacking vs consent/nav without cause.

### Files

| Path | Role |
|------|------|
| `web/src/components/experience/ReportReviewDialog.tsx` | UPDATE width classes |
| `web/src/components/experience/ReviewCard.tsx` | Host only |

### References

- PRD UX-4
- Story 12.4 report flow

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
