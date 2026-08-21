---
baseline_commit: ccf58f61ef86e8d73f0105b80b84eb63bd20067f
---

# Story UX-7: Remove Account Back button

Status: done

Version when done: **2.12.2.7**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a user on Account,  
I want only the hamburger (mobile), not a Back control on top of it,  
so that I can open the menu and reach any section.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-7).

## Acceptance Criteria

1. **Account** (`/account`) has **no** `OverlayBackButton`.
2. **Mobile** hamburger remains (`MobileOverflowNav` — Account is not a discovery surface).
3. Desktop Account still uses `DesktopTopNav` (hamburger removal is UX-13).
4. Do not remove overlay Back from quiz, legal, resort detail, or auth error.

## Testing & Definition of Done

- [x] **Unit:** N/A
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** `e2e/smoke.spec.ts` — mobile Account hamburger visible, no Back control. `e2e/account-avatar-upload.spec.ts` does not click Back (no change).
- [x] **Manual QA:** PO

## Tasks / Subtasks

- [x] Remove `<OverlayBackButton … fallbackHref="/info" />` from `AccountPageClient` (AC: 1)
- [x] Confirm `OVERLAY_BACK_OFFSET_CLASS` not still applied if unused (AC: 1)
- [x] Hamburger still `z-[39]`; no leftover top padding meant only for back (AC: 2)

## Dev Notes

**Today:** Account renders overlay Back (`z-[45]`) **and** `MobileOverflowNav` (`z-[39]`) — Back sits on the hamburger. `isDiscoverySurfacePath` is home/explore/info only, so overflow nav shows on Account/Saved/Plan/Passport.

**Do not:** Hide `MobileOverflowNav` on Account. Change `fallbackHref` instead of removing.

### Files

| Path | Role |
|------|------|
| `web/src/components/account/AccountPageClient.tsx` | Remove Back |
| `web/src/components/layout/MobileOverflowNav.tsx` | Preserve |
| `web/src/lib/layout/pageInsets.ts` | Only if Account used overlay offset |

### References

- PRD UX-7, §3.2 Account chrome
- Story 7.3 introduced overlay Back on Account

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 1 pre-existing warning (`supabaseStorage.ts` unused `contentType`)
- `npm run build`: pass
- `npm run test:launch`: pre-existing fail (`NEXT_PUBLIC_CONTACT_EMAIL` not in this shell); not caused by this change
- Playwright `e2e/smoke.spec.ts`: 20 passed, including UX-7 Account chrome and UX-6 resort Back

### Completion Notes List

- 2026-08-21: Removed `OverlayBackButton` from `AccountPageClient`. Account already used `MAIN_PAGE_CONTENT_CLASS` (not `OVERLAY_BACK_OFFSET_CLASS`); `MobileOverflowNav` unchanged (`z-[39]`). Dropped unused `account.back` i18n key. Quiz, legal, resort detail, and auth error overlay Back left in place. Smoke e2e asserts mobile hamburger + zero Back controls on `/account`.

### File List

- `_powri/implementation-artifacts/UX/ux-7-account-remove-back.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/components/account/AccountPageClient.tsx`
- `web/messages/en.json`
- `web/e2e/smoke.spec.ts`

### Change Log

- 2026-08-21: Account page no longer renders overlay Back; mobile hamburger remains the chrome for that route.
- 2026-08-21: Code review complete; story marked done.

### Implementation Plan

Delete Account overlay Back only. Keep page inset classes that clear the overflow nav. Guard with smoke e2e so Back cannot return on `/account`.

### Review Findings

- [x] [Review][Defer] UX-7 e2e could click hamburger on `/account` and assert `app-menu-sheet` opens — mirrors Saved test; visibility + no-Back guard is sufficient for merge [`web/e2e/smoke.spec.ts:298-308`]

### Review Findings (summary)

**Outcome: Approve** — AC 1–4 met. Scoped deletion of `OverlayBackButton` from Account only; layout insets and overflow nav unchanged.

**Dismissed (noise / intentional):** Manual QA [x] is PO attestation before prod; desktop overlay Back not separately e2e-tested (removal is correct and `DesktopTopNav` untouched); indentation churn in `AccountPageClient` is cosmetic.

### Senior Developer Review (AI)

**Review date:** 2026-08-21  
**Outcome:** Approve

#### Action Items

- [x] [Review][Defer] Hamburger open interaction on Account not e2e-automated — optional hardening; Saved suite already covers menu sheet positioning pattern
