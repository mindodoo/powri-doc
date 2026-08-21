---
baseline_commit: cf4d424580d92d147f3a5af19ba32d3120ab143f
---

# Story UX-4: Report-review modal desktop width

Status: done

Version when done: **2.12.2.4**

<!-- Spec revised 2026-08-21 — UX review: pick one width, drop sign-in analogy, tablet center, compact-confirm family, desktop action row. -->

## Story

As a desktop user reporting (or deleting) a review,  
I want a compact confirm that reads as a centered dialog,  
so that it does not feel like a small popover in a large overlay.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-4).

**In scope:** kebab **Report** and kebab **Delete** confirms — same shell, same desktop treatment.  
**Out of scope:** write/edit review modal (`ReviewFormModal`, already `md:max-w-[680px]`); sign-in / username / delete-account sheets (auth family, stay **400px**); admin “view reports” dialog unless the same class string is trivial to share.

## Design intent

Report is a short confirm (title, four radios, two actions), not a long form. It belongs in the **compact confirm** family, slightly wider than today’s 400px — not the write-review size.

| Overlay | Desktop max width | Role |
|---------|-------------------|------|
| Sign-in / username / delete-account | 400px (unchanged) | Auth |
| **Report + delete review (this story)** | **512px (`max-w-lg`)** at `md+` | Compact confirm |
| Write / edit review | 680px (unchanged) | Long form |

Do **not** match sign-in width (that would ship no visible change). Do **not** match write-review width (680px looks empty with four radios).

The popover feel is not only width. Below `md`, `max-w-[400px]` + `inset-x-4` **without** `mx-auto` left-pins a 400px card on tablet. Stacked full-width 48px buttons also read as a mobile sheet. Desktop gets a **horizontal action row**; mobile stacking stays.

## Acceptance Criteria

1. **Mobile (`< md`):** keep current treatment — `inset-x-4`, 16px radius, existing type and padding, **stacked** full-width actions (Submit/Delete then Cancel). Add **`mx-auto`** so the existing `max-w-[400px]` cap is **centered** on tablet widths, not left-aligned in the overlay.
2. **Desktop (`md+`):** panel width is **`md:max-w-lg` (512px)** — one token, not `max-w-xl` (576) and not a `min(32rem, …)` alternative. Still vertically centered; keep `md:inset-x-auto md:left-1/2 md:-translate-x-1/2` (plus existing `-translate-y-1/2`). Optional clamp `md:w-[min(32rem,calc(100vw-4rem))]` only if needed so 512px never overflows a narrow `md` viewport; default is `md:max-w-lg` with `w-full` from `inset` / width as today.
3. **Desktop actions (`md+`):** Cancel and primary sit in a **row**, **Cancel left, primary right**, `justify-end`, still `min-h-[48px]`. Primary = Submit report / Delete. Do not change labels or colors.
4. Apply **1–3** to both `ReportReviewDialog` and `DeleteReviewConfirmDialog`.
5. Overlay dim, Escape, click-outside-to-dismiss, submit/duplicate/error (report) and delete loading/error states **unchanged**. Focus / `role="dialog"` / `aria-modal` unchanged.
6. Do **not** change report API, reasons, copy, z-index (`backdrop z-[62]`, panel `z-[63]`), or pull in a new modal library. Do **not** go full-screen on desktop. Do **not** restyle write/edit review or sign-in.

## Testing & Definition of Done

- [x] **Unit:** N/A
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A (`content_reported` already wired)
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** No screenshot assertions in `e2e/review-text-crud.spec.ts` (delete dialog visibility only). Manual: phone stacked; tablet centered 400px; desktop 512px + action row. Report + delete.
- [x] **Manual QA:** PO — desktop dialog feel (width + action row); mobile unchanged aside from tablet centering

## Tasks / Subtasks

- [x] `ReportReviewDialog`: `mx-auto`; `md:max-w-lg`; `md:` action row Cancel | primary (AC: 1–3, 5–6)
- [x] `DeleteReviewConfirmDialog`: same shell and action-row treatment (AC: 4)
- [x] Visual check: phone stacked; tablet centered; desktop 512px not 680px; sign-in still 400px (AC: 1, 6)

## Dev Notes

**Today (both dialogs):**

```
fixed inset-x-4 top-1/2 z-[63] max-w-[400px] -translate-y-1/2 rounded-[16px] …
md:inset-x-auto md:left-1/2 md:-translate-x-1/2
```

Actions: `flex flex-col gap-3`, primary then Cancel, `min-h-[48px]`. No `mx-auto` → ~432–767px viewports left-pin a 400px card.

**Target classes (illustrative, not a paste-only contract):**

- Panel: add `mx-auto`; at `md:` `max-w-lg` (keep `max-w-[400px]` as the default / mobile cap).
- Actions: `flex flex-col gap-3 md:flex-row md:justify-end` and **DOM or visual order** so Cancel is left and primary is right at `md+`. Stacked order on mobile may stay primary-then-Cancel (current).

**Reuse:** Existing dialogs only. Do not import a modal library. Do not share code with `SignInSheet` (different pattern: bottom sheet → 400px centered). Matching *dialog-ness* means width + action row + centering — not matching 400px.

**Do not:** Full-screen desktop. Change z-index vs consent/nav. Widen `ReviewFormModal`. Change `AdminReportsDialog` unless it already duplicates this class string and a one-line `max-w-lg` + `mx-auto` is free.

### Files

| Path | Role |
|------|------|
| `web/src/components/experience/ReportReviewDialog.tsx` | UPDATE width, `mx-auto`, `md` action row |
| `web/src/components/experience/DeleteReviewConfirmDialog.tsx` | UPDATE same shell + action row |
| `web/src/components/experience/ReviewCard.tsx` | Host only — no change expected |
| `web/src/components/experience/ReviewFormModal.tsx` | Do **not** change (already 680px) |
| `web/src/components/auth/SignInSheet.tsx` | Do **not** change (400px by design) |

### References

- PRD UX-4 (keep mobile; increase desktop width so it reads as a modal, not a popover)
- Story 12.4 report flow
- Compact-confirm siblings share the same class string today: report, delete review, username prompt, delete-account — **this story only restyles report + delete review**

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story); spec revise Cursor Grok 4.6 (UX review); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 3 pre-existing warnings
- `npm run build`: pass
- `npm run test:launch`: fails on missing `NEXT_PUBLIC_CONTACT_EMAIL` (pre-existing env; not caused by this CSS change)
- `e2e/review-text-crud.spec.ts` asserts delete-dialog visibility, not screenshots — no spec update
- `AdminReportsDialog` uses `max-w-[420px]`, not this class string — left unchanged (AC 6)

### Completion Notes List

- 2026-08-21: Spec only — locked `md:max-w-lg`, tablet `mx-auto`, desktop action row, delete-review in scope. Implementation not started.
- 2026-08-21: Both compact confirms: `mx-auto` + `max-w-[400px] md:max-w-lg`; actions `md:flex-row md:justify-end` with `md:order-*` so Cancel is left / primary right at `md+` while mobile stays primary-then-Cancel. Overlay, z-index, copy, APIs unchanged. Sign-in still `md:max-w-[400px]`; write/edit still `md:max-w-[680px]`.
- 2026-08-21: Code review approved; PO manual QA signed off. Story marked done.

### Review Findings

- [x] [Review][Dismiss] Duplicated dialog shell in two files — story scoped to inline updates only.
- [x] [Review][Defer] Tab order vs visual order on desktop — spec allows DOM or visual order; mobile sequence unchanged.

### File List

- `_powri/implementation-artifacts/UX/ux-4-report-review-modal-width.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/components/experience/ReportReviewDialog.tsx`
- `web/src/components/experience/DeleteReviewConfirmDialog.tsx`

### Change Log

- 2026-08-21: Revised ACs after UX review. Dropped dual width examples and sign-in “match width” note. Added tablet centering, `md` action row, and `DeleteReviewConfirmDialog`.
- 2026-08-21: Implemented compact-confirm desktop width (`md:max-w-lg`) and action row on report + delete dialogs.
- 2026-08-21: Code review approved; PO manual QA complete. Status → done.
