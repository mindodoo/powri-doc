---
baseline_commit: ca4dc42c47bad76222d56189a0af7a84185a348b
---

# Story UX-6: Resort detail Back → Explore

Status: done

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

- [x] **Unit:** N/A
- [x] **Quiz / scoring:** N/A — do not change `QuizFlow` OverlayBackButton
- [x] **Analytics:** N/A
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** `e2e/smoke.spec.ts` — direct URL Back is `/explore`; Home → resort → Back = Explore (not Home). Review-submit path is the same `href="/explore"` link (not `router.back()`).
- [x] **Manual QA:** PO

## Tasks / Subtasks

- [x] `ResortDetailOverlayBack`: pass `href="/explore"` (AC: 1–2, 4)
- [x] Remove history `fallbackHref="/"` on this instance only (AC: 5)

### Review Findings

- [x] [Review][Defer] Review-submit → Back e2e not automated — AC2 satisfied by `href="/explore"` link semantics; Supabase session mint flaky in local e2e (same as `review-text-crud`)

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

Cursor Grok 4.6 (create-story); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 1 pre-existing warning (`supabaseStorage.ts` unused `contentType`)
- `npm run build`: pass
- `npm run test:launch`: pre-existing fail (`NEXT_PUBLIC_CONTACT_EMAIL`); not caused by this change
- Playwright (`env -u PLAYWRIGHT_BROWSERS_PATH`): UX-6 smoke tests pass; remaining guest/smoke e2e pass; `account-avatar-upload` / `review-text-crud` fail on Supabase “Database error creating new user” (pre-existing env, not this story)

### Completion Notes List

- 2026-08-21: `ResortDetailOverlayBack` now uses `OverlayBackButton` link mode (`href="/explore"`, `variant="onImage"`, `ariaLabel={t('back')}`). History `fallbackHref="/"` removed on this instance only. `OverlayBackButton` dual API, `QuizFlow` (`href="/explore"`), legal (`href="/info"`), and Account (`fallbackHref="/info"`) unchanged. Smoke e2e covers direct URL and Home → detail → Back → Explore.

### File List

- `_powri/implementation-artifacts/UX/ux-6-resort-back-to-explore.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/components/resort/ResortDetailOverlayBack.tsx`
- `web/e2e/smoke.spec.ts`

### Change Log

- 2026-08-21: Resort detail overlay Back is a link to `/explore` instead of history back.
- 2026-08-21: Code review complete; story marked done.

### Implementation Plan

Use existing `OverlayBackButton` `href` branch (same as quiz/legal). Do not change history-mode default. Guard with smoke e2e so Home → detail no longer returns Home.

### Review Findings (summary)

**Outcome: Approve** — AC 1–5 met. One-line swap to link mode is the correct fix; scope stays on `ResortDetailOverlayBack` only.

**Dismissed (noise / intentional):** Explore filter state not preserved on Back (PRD requires fixed `/explore`); `next/link` vs locale Link matches existing `OverlayBackButton` consumers; aria stays `resort.back` per AC4.

### Senior Developer Review (AI)

**Review date:** 2026-08-21  
**Outcome:** Approve

#### Action Items

- [x] [Review][Defer] Review-submit → Back e2e not automated — AC2 covered by implementation; optional when Supabase e2e is stable
