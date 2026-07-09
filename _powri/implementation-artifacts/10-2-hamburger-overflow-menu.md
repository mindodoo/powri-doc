---
baseline_commit: 37f9aa2f41738bd9e3aa4386e99a9a092721a833
---

# Story 10.2: Unified Desktop Nav & Hamburger Overflow

Status: done

<!-- PO additions 2026-07-03 — extends epic Story 10.2 with desktop IA redesign and Info→About rename -->
<!-- PO follow-up 2026-07-09 — UX polish: page title alignment, remove eyebrows, desktop nav edge layout (PO approved 2026-07-09) -->

## Story

As a **visitor on any device**,
I want navigation that matches how I browse on mobile vs desktop,
So that primary actions stay thumb-friendly on mobile while desktop feels like a clean single top bar.

**Source shard(s):** [`epics/phase2/shards/epic-10-nav-saved.md`](../../planning-artifacts/epics/phase2/shards/epic-10-nav-saved.md) · [`prds/phase2/shards/12-features-navigation.md`](../../planning-artifacts/prds/phase2/shards/12-features-navigation.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **10.1** (adaptive nav baseline, `navConfig.ts`, `DesktopTopNav`, stub routes)

---

## Acceptance Criteria

### A. Mobile hamburger overflow (<768px) — epic baseline

1. **Given** viewport **< 768px**  
   **When** I open the hamburger  
   **Then** I see **Passport**, **About** (renamed from Info), **Sign in / Account** — **not** Home / Explore / Saved / Plan (no duplication with bottom nav)

2. **And** Passport shows badge count chip when user has ≥1 badge  
3. **And** `nav_menu_opened` retained from Phase 1  
4. **And** mobile bottom nav remains **Home · Explore · Saved · Plan** (unchanged from 10.1)

### B. Desktop single top navigation (≥768px) — PO addition

5. **Given** viewport **≥768px**  
   **When** I view any main page  
   **Then** I see **one** top navigation bar only — **no** second stacked row (hide Phase 1 `DiscoveryTopBar` at `md+`)

6. **And** the desktop bar layout is (left → right):

   | Zone | Content |
   |------|---------|
   | Far left | Hamburger icon (opens overflow menu) |
   | Left | **Powri** wordmark text (not a duplicate of Home link styling; appropriate gap after hamburger) |
   | Centre | **Home · Explore · Plan · About** — spaced for wide screens |
   | Right cluster | Search icon · Sign in **or** profile avatar (far right) |

7. **And** desktop hamburger overflow contains **Saved** and **Passport** only (profile-specific / secondary on desktop web)

8. **And** **Saved** and **Passport** are **not** in the desktop centre nav links (they live in hamburger on desktop)

9. **And** Sign in shows when guest; profile picture/avatar links to `/account` when authenticated (same auth patterns as 10.1)

10. **And** Search icon opens inline search panel (reuse `ResortSearchField` / home search pattern; `listContext` appropriate to current page)

### C. Info → About rename — PO addition

11. **Given** any nav surface (bottom nav unaffected; desktop top nav, mobile hamburger, routes unchanged)  
    **Then** user-facing label **"Info"** is replaced with **"About"** in navigation (`nav.about` i18n key; route stays `/info`)

12. **And** page titles/eyebrows on `/info` may use "About" where nav label appears; legal page content headings unchanged unless already "About this app"

### D. Analytics & regression

13. **And** `nav_destination_tapped` fires with `nav_type: 'hamburger'` for hamburger menu item taps (desktop + mobile overflow)  
14. **And** desktop search open retains / extends existing search analytics where applicable  
15. **And** no invalid HTML nesting (`<a>` inside `<a>`) in nav or card components

### E. UX polish — PO follow-up (2026-07-09)

*Blocks story completion until implemented and PO approves visual QA. **Do not** update E2E/smoke tests or run full regression until PO sign-off on the fix.*

16. **Given** I navigate between **Explore**, **Plan**, and **About** (`/info`) on the same viewport  
    **Then** each page’s main `<h1>` title starts at the **same horizontal and vertical position** (consistent top inset and left edge within `PageContent` / `max-w-content`)

17. **And** page-title **eyebrows** (small caption above `<h1>`, e.g. “Explore”, “Plan”, “About”) are **removed** on all main AppShell routes — including Saved, Passport, and Account — without removing in-content section labels (e.g. card headings, “HOW PEOPLE EXPERIENCED IT”)

18. **Given** viewport **≥768px**  
    **When** I view any page (especially Home with full-bleed hero)  
    **Then** the desktop nav **hamburger** and **Powri** wordmark sit in a **left cluster pinned to the viewport’s leading edge** (with standard horizontal gutter), **not** grouped inside the centred `max-w-content` column with the nav links

19. **And** on desktop the **search icon** and **Sign in / avatar** sit in a **right cluster pinned to the viewport’s trailing edge** (same gutter as hero/content), aligning with full-width home hero layout

20. **And** centre nav links (**Home · Explore · Plan · About**) remain visually centred in the space **between** the left and right edge clusters (IA unchanged from 10.2)

21. **And** the full-width nav bottom border (`border-divider`) does not create a visually awkward “narrow band” effect against the edge-aligned chrome — adjust or remove border treatment on desktop if needed so the bar feels coherent with full-bleed hero

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [x] **Unit (Vitest):** `navConfig` split — desktop centre items vs hamburger items vs mobile bottom items
- [x] **Analytics:** `nav_type: 'hamburger'` wired; `phase2Events.ts` + `npm run test:analytics`
- [x] **User-facing flow:** Update `web/e2e/smoke.spec.ts` — About via desktop top nav or mobile hamburger; no double top nav at 1024px
- [x] **PR gates:** `lint`, `build`, `test:launch`, `test:e2e`
- [x] **Manual QA:** 375px hamburger → Passport, About, Account; 1024px single bar with hamburger+Powri+centre links+search+auth; desktop hamburger → Saved, Passport only

### Follow-up UX polish (PO 2026-07-09)

- [x] **Visual QA (PO):** Explore / Plan / About title alignment; desktop nav flush left/right on home hero at 1280px+; no page-title eyebrows — **approved 2026-07-09**
- [x] **PR gates (after PO approval):** `lint`, `build`, `test:unit`, `test:analytics` pass; `test:launch` fails locally on missing `NEXT_PUBLIC_CONTACT_EMAIL` (pre-existing env)
- [x] **E2E / smoke:** Re-ran `test:e2e` — 9/9 pass; added h1 alignment regression test in `smoke.spec.ts`

---

## Tasks / Subtasks

- [x] **Refactor `navConfig.ts`** (AC: 4, 6–8, 11)
  - [x] `MOBILE_BOTTOM_NAV_ITEMS` — Home, Explore, Saved, Plan
  - [x] `DESKTOP_CENTRE_NAV_ITEMS` — Home, Explore, Plan, About (`/info`)
  - [x] `DESKTOP_HAMBURGER_ITEMS` — Saved, Passport
  - [x] `MOBILE_HAMBURGER_ITEMS` — Passport, About, Account/Sign in
  - [x] Rename `labelKey` `info` → `about` in nav config (route `/info` unchanged)

- [x] **Replace `DesktopTopNav` with unified bar** (AC: 5–10)
  - [x] Merge hamburger + Powri brand + centre links + search + auth into one component (or compose subcomponents)
  - [x] Hamburger opens `AppMenuSheet` variant with context-aware items (`desktop` vs `mobile` item lists)
  - [x] Hide/remove duplicate `DiscoveryTopBar` at `md+` on home/explore/info — search moves to desktop top nav icon
  - [x] Revert `DISCOVERY_PAGE_TOP_OFFSET_CLASS` / `md:top-14` offsets from 10.1 interim two-row layout where no longer needed

- [x] **`AppMenuSheet` overflow dedupe** (AC: 1–3, 7)
  - [x] Mobile: Passport, About, Sign in/Account only
  - [x] Desktop: Saved, Passport only (opened from desktop hamburger)
  - [x] Passport badge count chip when ≥1 badge (stub hidden until Story 13.x — PO confirmed)
  - [x] Remove Home/Explore/Saved/Plan/About from all hamburger menus

- [x] **i18n — Info → About** (AC: 11–12)
  - [x] `nav.info` → `nav.about`: "About"
  - [x] Update references in components, e2e, tracking-plan property values if `destination: 'info'` → `'about'` (coordinate analytics)

- [x] **Analytics** (AC: 13–14)
  - [x] Hamburger menu item taps → `nav_destination_tapped` with `nav_type: 'hamburger'`
  - [x] Update `phase2Events.ts` instrumentation files for `AppMenuSheet` / unified desktop nav

- [x] **E2E & layout QA** (AC: 5, 15)
  - [x] Smoke test: About navigation; assert single top nav at 1280px
  - [x] Verify Saved/Plan/Passport stub pages render without console hydration errors

- [x] **Page header consistency — title alignment** (AC: 16)
  - [x] Audit top offsets on `/explore`, `/plan`, `/info` — unify desktop + mobile insets so `<h1>` baseline matches
  - [x] Resolve structural drift: Explore/About use discovery offsets + optional subtitle; Plan uses `MOBILE_OVERFLOW_PAGE_OFFSET_CLASS` only
  - [x] Prefer shared header shell (e.g. reuse `PageContent` + shared `PageHeader` or aligned offset constants in `pageInsets.ts`) over per-page one-offs

- [x] **Remove page-title eyebrows** (AC: 17)
  - [x] Remove eyebrow `<p>` from: `explore/page.tsx`, `plan/page.tsx`, `info/page.tsx`, `saved/page.tsx`, `passport/page.tsx`, `AccountPageClient.tsx`
  - [x] Remove unused `*.eyebrow` keys from `web/messages/en.json` (or leave keys orphaned if safer for i18n tooling — document choice)
  - [x] **Out of scope:** in-content section eyebrows / badge typography (e.g. `SectionHeader`, review blocks)

- [x] **Desktop nav edge layout** (AC: 18–21)
  - [x] Refactor `DesktopTopNav.tsx`: three-zone grid — left (☰ + Powri) | centre (nav links) | right (🔍 + auth) — **full viewport width**, not single `max-w-content` flex row
  - [x] Apply `px-screen-x` (or match hero gutter) on outer nav shell; centre links centred in remaining space
  - [x] Revisit `border-b border-divider` on `<nav>` — PO reports full-width rule looks odd against hero; consider border only on inner row, transparent nav on home until scroll, or remove on desktop
  - [x] Search dropdown panel: keep full-width or align panel inset with content column (PO preference: utilities at viewport edge; search panel can span content width)

- [x] **PO visual sign-off** (gate)
  - [x] PO confirms fix on Home (hero), Explore, Plan, About at ≥1280px and 375px (2026-07-09)
  - [x] E2E smoke re-run + h1 alignment test added

---

## Dev Notes

### PO desktop IA (authoritative for 10.2)

```
Desktop (≥768px) — ONE bar (viewport-edge utilities, centred links):
[☰ Powri]                    Home · Explore · Plan · About                    [🔍 Sign in|Avatar]
↑ viewport leading edge                                                          ↑ viewport trailing edge

Desktop hamburger (☰):
  Saved
  Passport  (badge chip if ≥1)

Mobile (<768px) — unchanged bottom nav from 10.1:
Bottom: Home · Explore · Saved · Plan

Mobile hamburger (☰):
  Passport  (badge chip)
  About
  Sign in / Account
```

**10.2 implementation note:** First pass placed hamburger + utilities inside `max-w-content`, which PO flagged as misaligned with full-bleed home hero — follow-up AC 18–21 corrects this.

**Rationale:** Saved and Passport are profile/collection features — discoverable in bottom nav on mobile (thumb zone) but tucked in hamburger on desktop to keep the centre nav focused on browse + plan flows.

### Follow-up UX polish — PO feedback (2026-07-09)

| # | Symptom | Root cause (current) | Fix direction |
|---|---------|----------------------|---------------|
| 1 | Explore / Plan / About titles feel misaligned | Different page shells: discovery offsets vs overflow offset; eyebrow + subtitle presence varies | Shared page header + unified top padding; remove eyebrows (#2) |
| 2 | Eyebrows add space without benefit | Caption line above `<h1>` on stub/discovery pages | Remove page-title eyebrows site-wide (not in-content section labels) |
| 3 | Desktop hamburger feels “centred” with nav links | `DesktopTopNav` inner row is `mx-auto max-w-content` — all controls in content column | Left cluster pinned to viewport leading edge |
| 4 | Search + auth not aligned with hero right edge | Same `max-w-content` constraint as #3 | Right cluster pinned to viewport trailing edge |

**Primary files:** `DesktopTopNav.tsx`, `pageInsets.ts`, `PageContent.tsx` (optional shared header), page routes listed in tasks, `web/messages/en.json`.

**Test gate (PO directive):** Implement and obtain PO visual approval **before** running `test:e2e` or editing `web/e2e/smoke.spec.ts`.

### What 10.1 left as interim (fix in 10.2)

| Interim (10.1) | Target (10.2) |
|----------------|----------------|
| Two stacked top bars on desktop (DesktopTopNav + DiscoveryTopBar) | Single unified top bar |
| Saved/Plan/Passport in desktop centre nav | Saved + Passport → desktop hamburger only |
| Info label | About label |
| Home/Explore duplicated in mobile hamburger | Removed from hamburger |
| `nav_type: 'hamburger'` not instrumented | Wire in 10.2 |

### Reuse from 10.1 (do not reinvent)

| Pattern | Location |
|---------|----------|
| Nav config / analytics | `navConfig.ts`, `navAnalytics.ts` |
| Auth in nav | `AuthProvider`, `resolveAvatarSrc`, `openSignIn({ trigger: 'nav' })` |
| Search field | `ResortSearchField`, `DiscoveryTopBar` search panel logic |
| Mobile bottom nav | `BottomNav.tsx` — keep 4-tab; only label/route config updates for About N/A on bottom |

### Analytics destination rename

If renaming `destination: 'info'` → `'about'`, update:
- `navConfig.ts` destination enum
- `docs/analytics/tracking-plan.md` §3 (note alias or breaking change — prefer `'about'` for new events; document if `'info'` deprecated)

### Out of scope (10.2)

- Save heart / API (Story 10.3)
- Full `/saved` page (Story 10.4)
- Passport badge evaluation (Story 13.x)
- PRD shard regen (update story + implementation; PM can regen monolith later)

### References

- [Source: epics/phase2/epics-phase2.md — Story 10.2]
- [Source: prds/phase2/shards/12-features-navigation.md — FR-16.3]
- [Source: ux-phase2/experience.md — desktop single top nav]
- [Source: implementation-artifacts/10-1-adaptive-navigation-mobile-desktop.md — interim two-row note]

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- PO clarifications (2026-07-09): badge chip stub hidden; analytics destination stays `info`; global mobile hamburger via `MobileOverflowNav`; desktop search `listContext: 'explore'`
- Fixed AppMenuSheet z-index (z-56) above DesktopTopNav (z-50) so desktop hamburger links are clickable

### Completion Notes List

- Unified `DesktopTopNav`: hamburger + Powri wordmark + centre links + search + auth; desktop search uses `listContext: 'explore'`
- `MobileOverflowNav` in AppShell for non-discovery mobile routes (Saved, Plan, Passport, Account, etc.)
- Discovery pages keep hamburger in `DiscoveryTopBar` on mobile; `DiscoveryTopBar` hidden at `md+`
- `navConfig` split into MOBILE_BOTTOM / DESKTOP_CENTRE / DESKTOP_HAMBURGER / MOBILE_HAMBURGER item lists
- `navAnalytics` extended with `hamburger` nav_type; `AppMenuSheet` wires `nav_destination_tapped`
- i18n: `nav.about` label; `/info` page eyebrow → "About"; analytics destination remains `info`
- Unit: `navConfig.test.ts`; integration: `navAnalytics.test.ts`; E2E: 3 new Story 10.2 smoke cases (8/8 pass)

- i18n: removed unused page-title `*.eyebrow` keys from `en.json` (not orphaned)

### File List (follow-up UX polish 2026-07-09)

- `web/src/components/layout/PageHeader.tsx` (added)
- `web/src/components/layout/DesktopTopNav.tsx` (modified — viewport-edge grid, no nav border-b)
- `web/src/lib/layout/pageInsets.ts` (modified — `MAIN_PAGE_*` shell constants)
- `web/src/lib/layout/pageInsets.test.ts` (modified)
- `web/src/app/[locale]/explore/page.tsx` (modified)
- `web/src/app/[locale]/plan/page.tsx` (modified)
- `web/src/app/[locale]/info/page.tsx` (modified)
- `web/src/app/[locale]/saved/page.tsx` (modified)
- `web/src/app/[locale]/passport/page.tsx` (modified)
- `web/src/components/account/AccountPageClient.tsx` (modified)
- `web/messages/en.json` (modified — removed page-title eyebrow keys)
- `web/e2e/smoke.spec.ts` (modified — h1 alignment regression test)

### File List (Story 10.2 initial implementation)

- `web/src/components/layout/navConfig.ts` (modified)
- `web/src/components/layout/navConfig.test.ts` (modified)
- `web/src/components/layout/navAnalytics.ts` (modified)
- `web/src/components/layout/navAnalytics.test.ts` (added)
- `web/src/components/layout/AppMenuSheet.tsx` (modified)
- `web/src/components/layout/DesktopTopNav.tsx` (modified)
- `web/src/components/layout/MobileOverflowNav.tsx` (added)
- `web/src/components/layout/AppShell.tsx` (modified)
- `web/src/components/layout/DiscoveryTopBar.tsx` (modified)
- `web/src/lib/layout/pageInsets.ts` (modified)
- `web/src/lib/layout/pageInsets.test.ts` (modified)
- `web/src/lib/analytics/phase2Events.ts` (modified)
- `web/messages/en.json` (modified)
- `web/src/app/[locale]/saved/page.tsx` (modified)
- `web/src/app/[locale]/plan/page.tsx` (modified)
- `web/src/app/[locale]/passport/page.tsx` (modified)
- `web/src/components/account/AccountPageClient.tsx` (modified)
- `web/e2e/smoke.spec.ts` (modified)

## Senior Developer Review (AI)

**Outcome:** Approve (with notes)  
**Date:** 2026-07-09

### Summary

Implementation satisfies all ACs per PO clarifications. Tests pass (`lint`, `build`, `test:unit`, `test:analytics`, `test:e2e` smoke 8/8). `test:launch` fails locally only on missing `NEXT_PUBLIC_CONTACT_EMAIL` (pre-existing env gap).

### Action Items

- [x] [Medium] **Follow-up AC 16–21** — page title alignment, remove eyebrows, desktop nav viewport-edge layout (PO 2026-07-09) — **done, PO approved**
- [ ] [Low] Consider `useMediaQuery` to omit `DiscoveryTopBar` from DOM at `md+` instead of CSS-only hide (a11y tree still contains hidden header)
- [ ] [Low] Legal/resort-detail mobile pages lack `MobileOverflowNav` top padding — acceptable for 10.2 scope; revisit if those routes need hamburger reachability

## Change Log

- 2026-07-09: Story marked done after PO sign-off; PR raised
- 2026-07-09: PO approved follow-up UX polish; E2E smoke 9/9 pass (added h1 alignment test)
- 2026-07-09: Follow-up UX polish implemented — aligned page headers, removed eyebrows, desktop nav viewport-edge layout; awaiting PO visual sign-off before E2E
- 2026-07-09: Story 10.2 implemented — unified desktop nav, hamburger overflow dedupe, Info→About label, analytics hamburger nav_type, global mobile overflow nav, tests
