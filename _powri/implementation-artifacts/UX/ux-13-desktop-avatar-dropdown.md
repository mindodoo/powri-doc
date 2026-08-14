# Story UX-13: Desktop avatar dropdown (no hamburger)

Status: ready-for-dev

Version when done: **2.12.2.13**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a desktop user,  
I want Account, Saved, and Passport under my profile picture,  
so that the hamburger is gone and account destinations live in one menu.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-13, D4, D6, §3.1).

## Acceptance Criteria

1. **Desktop (`md+`):** **no hamburger**. Do not render `AppMenuSheet` `variant="desktop"` or the Menu button in `DesktopTopNav`.
2. **Signed in:** profile picture is a **dropdown** (not a direct `/account` link) with **Account**, **Saved**, **Passport**. Click-outside and Escape close it.
3. **Signed out:** **Sign in** stays in the top bar (`openSignIn({ trigger: 'nav' })`). **Saved and Passport hidden** until signed in (D4).
4. Centre desktop links unchanged: Home, Explore, Plan, About (`DESKTOP_CENTRE_NAV_ITEMS`).
5. **Mobile hamburger unchanged** (`MOBILE_HAMBURGER_ITEMS`, `DiscoveryTopBar` / `MobileOverflowNav`).
6. Nav analytics: dropdown items fire `nav_destination_tapped` (`destination`: `account` \| `saved` \| `passport`). Reuse `nav_type: 'top'` **or** extend `NavType` with `'avatar'` — if extended, update `phase2Events` + `docs/analytics/tracking-plan.md` in the **same PR**.
7. Keyboard: avatar button `aria-expanded` / `aria-haspopup="menu"`; items are links.

## Testing & Definition of Done

- [ ] **Unit (Vitest):** `navConfig.test.ts` — desktop hamburger list unused or empty; mobile items unchanged
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** Only if new `nav_type`; else existing event
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual desktop signed-in/out. Update `navConfig.test.ts` “desktop hamburger overflow contains Saved and Passport only”
- [ ] **Manual QA:** PO

## Tasks / Subtasks

- [ ] Remove desktop Menu + `AppMenuSheet` desktop instance (AC: 1)
- [ ] Avatar menu component (AC: 2, 7)
- [ ] Signed-out Sign in; no Saved/Passport (AC: 3)
- [ ] Update `navConfig.test.ts` (AC: 4–5)
- [ ] Analytics if `NavType` changes (AC: 6)

## Dev Notes

**Today:** `DesktopTopNav` left: Menu + brand text; right: search icon + `Link href="/account"` avatar **or** Sign in. `DESKTOP_HAMBURGER_ITEMS` = Saved, Passport. `AppMenuSheet variant="desktop"`.

**Reuse:** `NavAvatar`, `trackNavDestination`, `resolveAvatarSrc`. Pattern from `AppMenuSheet` links — do **not** keep a second desktop sheet.

**Coordinate with UX-10:** Logo replaces brand text in the left cluster; without hamburger the left cluster is **logo only**.

**Do not:** Remove mobile hamburger. Put Saved in `DESKTOP_CENTRE_NAV_ITEMS`.

### Files

| Path | Role |
|------|------|
| `web/src/components/layout/DesktopTopNav.tsx` | UPDATE |
| `web/src/components/layout/navConfig.ts` | Desktop hamburger may become unused |
| `web/src/components/layout/navConfig.test.ts` | UPDATE |
| `web/src/components/layout/AppMenuSheet.tsx` | Desktop variant unused |
| `web/src/components/layout/navAnalytics.ts` | Optional `avatar` type |
| `web/messages/en.json` `nav.*` | Labels |

### References

- PRD UX-13, D4, D6, §3.1
- Story 10.2 hamburger overflow

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
