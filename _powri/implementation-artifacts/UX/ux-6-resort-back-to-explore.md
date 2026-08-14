# Story UX-6: Resort detail Back → Explore

Status: ready-for-dev

Version when done: **2.12.2.6**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a skier leaving a resort page,  
I want Back to always open Explore,  
so that finishing a review never sends me into the review form again.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-6).

## Acceptance Criteria

1. Resort detail overlay Back is a **link to `/explore`**, not `router.back()`.
2. After write/edit review (`ReviewPageClient` pushes `/resorts/[slug]`), tapping Back goes to **Explore**, not `/resorts/[slug]/review`.
3. Quiz, legal, and Account Back behaviour **unchanged** in this story (Account Back is UX-7).
4. Visual: keep `onImage` variant, 44×44, i18n `resort.back`.
5. Fallback when history is empty is irrelevant if using `href="/explore"`.

## Testing & Definition of Done

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A — do not change `QuizFlow` OverlayBackButton
- [ ] **Analytics:** N/A
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual: Home → resort → Back = Explore. Review submit → detail → Back = Explore. `e2e/smoke.spec.ts` if it asserts history back
- [ ] **Manual QA:** PO

## Tasks / Subtasks

- [ ] `ResortDetailOverlayBack`: pass `href="/explore"` (AC: 1–2, 4)
- [ ] Remove history `fallbackHref="/"` on this instance only (AC: 5)

## Dev Notes

**Today:** `ResortDetailOverlayBack` uses history mode (`fallbackHref="/"`) → `OverlayBackButton` `router.back()` when `history.length > 1`. Review flow: `ExperienceSectionClient` `router.push(/resorts/slug/review)` then `ReviewPageClient` `router.push(/resorts/slug)` — Back returns to review. Story 7.3 AC was “Detail → history back / `/`” — **this story overrides that for detail only**.

**Reuse:** `OverlayBackButton` already supports `href` (quiz/legal). Do **not** change the history helper’s default.

### Files

| Path | Role |
|------|------|
| `web/src/components/resort/ResortDetailOverlayBack.tsx` | UPDATE `href="/explore"` |
| `web/src/components/layout/OverlayBackButton.tsx` | Preserve dual API |
| `web/src/app/[locale]/resorts/[slug]/page.tsx` | Host |

### References

- PRD UX-6
- Story 7.3 overlay back (detail behaviour changes)

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
