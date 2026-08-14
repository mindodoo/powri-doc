# Story UX-7: Remove Account Back button

Status: ready-for-dev

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

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** N/A
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual mobile Account — hamburger tappable, no back overlay. `e2e/account-avatar-upload.spec.ts` if it clicks back
- [ ] **Manual QA:** PO

## Tasks / Subtasks

- [ ] Remove `<OverlayBackButton … fallbackHref="/info" />` from `AccountPageClient` (AC: 1)
- [ ] Confirm `OVERLAY_BACK_OFFSET_CLASS` not still applied if unused (AC: 1)
- [ ] Hamburger still `z-[39]`; no leftover top padding meant only for back (AC: 2)

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

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
