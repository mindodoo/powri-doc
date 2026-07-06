# Story 10.2: Unified Desktop Nav & Hamburger Overflow

Status: backlog

<!-- PO additions 2026-07-03 — extends epic Story 10.2 with desktop IA redesign and Info→About rename -->

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

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [ ] **Unit (Vitest):** `navConfig` split — desktop centre items vs hamburger items vs mobile bottom items
- [ ] **Analytics:** `nav_type: 'hamburger'` wired; `phase2Events.ts` + `npm run test:analytics`
- [ ] **User-facing flow:** Update `web/e2e/smoke.spec.ts` — About via desktop top nav or mobile hamburger; no double top nav at 1024px
- [ ] **PR gates:** `lint`, `build`, `test:launch`, `test:e2e`
- [ ] **Manual QA:** 375px hamburger → Passport, About, Account; 1024px single bar with hamburger+Powri+centre links+search+auth; desktop hamburger → Saved, Passport only

---

## Tasks / Subtasks

- [ ] **Refactor `navConfig.ts`** (AC: 4, 6–8, 11)
  - [ ] `MOBILE_BOTTOM_NAV_ITEMS` — Home, Explore, Saved, Plan
  - [ ] `DESKTOP_CENTRE_NAV_ITEMS` — Home, Explore, Plan, About (`/info`)
  - [ ] `DESKTOP_HAMBURGER_ITEMS` — Saved, Passport
  - [ ] `MOBILE_HAMBURGER_ITEMS` — Passport, About, Account/Sign in
  - [ ] Rename `labelKey` `info` → `about` in nav config (route `/info` unchanged)

- [ ] **Replace `DesktopTopNav` with unified bar** (AC: 5–10)
  - [ ] Merge hamburger + Powri brand + centre links + search + auth into one component (or compose subcomponents)
  - [ ] Hamburger opens `AppMenuSheet` variant with context-aware items (`desktop` vs `mobile` item lists)
  - [ ] Hide/remove duplicate `DiscoveryTopBar` at `md+` on home/explore/info — search moves to desktop top nav icon
  - [ ] Revert `DISCOVERY_PAGE_TOP_OFFSET_CLASS` / `md:top-14` offsets from 10.1 interim two-row layout where no longer needed

- [ ] **`AppMenuSheet` overflow dedupe** (AC: 1–3, 7)
  - [ ] Mobile: Passport, About, Sign in/Account only
  - [ ] Desktop: Saved, Passport only (opened from desktop hamburger)
  - [ ] Passport badge count chip when ≥1 badge
  - [ ] Remove Home/Explore/Saved/Plan/About from all hamburger menus

- [ ] **i18n — Info → About** (AC: 11–12)
  - [ ] `nav.info` → `nav.about`: "About"
  - [ ] Update references in components, e2e, tracking-plan property values if `destination: 'info'` → `'about'` (coordinate analytics)

- [ ] **Analytics** (AC: 13–14)
  - [ ] Hamburger menu item taps → `nav_destination_tapped` with `nav_type: 'hamburger'`
  - [ ] Update `phase2Events.ts` instrumentation files for `AppMenuSheet` / unified desktop nav

- [ ] **E2E & layout QA** (AC: 5, 15)
  - [ ] Smoke test: About navigation; assert single top nav at 1280px
  - [ ] Verify Saved/Plan/Passport stub pages render without console hydration errors

---

## Dev Notes

### PO desktop IA (authoritative for 10.2)

```
Desktop (≥768px) — ONE bar:
[☰]  Powri     Home · Explore · Plan · About          [🔍] [Sign in | Avatar]

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

**Rationale:** Saved and Passport are profile/collection features — discoverable in bottom nav on mobile (thumb zone) but tucked in hamburger on desktop to keep the centre nav focused on browse + plan flows.

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

_(filled on implementation)_

### Debug Log References

### Completion Notes List

### File List
