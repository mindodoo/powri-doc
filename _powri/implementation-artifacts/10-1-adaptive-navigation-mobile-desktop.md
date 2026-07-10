---
baseline_commit: d1daace38388edbc187d00336799c40a04101894
---

# Story 10.1: Adaptive Navigation — Mobile Bottom + Desktop Top

Status: done

<!-- Validation optional: bmad-create-story before bmad-dev-story -->

## Story

As a **visitor on any device**,
I want clear navigation to Home, Explore, Saved, and Plan,
So that I find Phase 2 features without hunting menus.

**Source shard(s):** [`epics/phase2/shards/epic-10-nav-saved.md`](../../planning-artifacts/epics/phase2/shards/epic-10-nav-saved.md) · [`prds/phase2/shards/12-features-navigation.md`](../../planning-artifacts/prds/phase2/shards/12-features-navigation.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

## Acceptance Criteria

1. **Given** viewport **< 768px**  
   **When** I view any main page wrapped in `AppShell`  
   **Then** `BottomNav` shows **Home · Explore · Saved · Plan** (4 items, fixed, always visible — never hide on scroll)

2. **Given** viewport **≥ 768px** (`md` breakpoint)  
   **When** I view any main page  
   **Then** bottom nav is **hidden** and **DesktopTopNav** shows: Home · Explore · Saved · Plan · Passport · Info · Sign in / Avatar

3. **And** active tab/link styling matches Theme A: accent label colour + dot indicator on mobile bottom nav; accent text on desktop top nav for current route

4. **And** tapping any nav destination fires `nav_destination_tapped` with:
   - `destination`: `home` | `explore` | `saved` | `plan` | `passport` | `info` | `account` | `sign_in`
   - `nav_type`: `bottom` | `top` (desktop Sign in uses `top`; do not use `hamburger` in 10.1 — that is Story 10.2)

5. **And** stub routes exist so nav links never 404: `/saved`, `/plan`, `/passport` (minimal placeholder pages acceptable until Stories 10.4 / 14.2 / 13.3)

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [x] **Analytics:** Wire `nav_destination_tapped` in nav components; update `phase2Events.ts` `instrumentationFiles` for the event; `npm run test:analytics` (registry + tracking-plan sync only — Phase 2 instrumentation gate is Epic 15.2)
- [x] **User-facing flow:** Update `web/e2e/smoke.spec.ts` — Info is no longer in bottom nav (use hamburger on mobile or desktop top nav for Info step)
- [x] **PR gates:** `lint`, `build`, `test:launch`, `test:e2e`
- [x] **Manual QA:** 375px — 4 bottom tabs visible, Info absent from bottom bar; 1024px — top nav only, no bottom nav; active states correct on `/`, `/explore`, `/saved`, `/plan`

## Tasks / Subtasks

- [x] **Shared nav config** (AC: 1, 2, 3)
  - [x] Create `web/src/components/layout/navConfig.ts` — single source for href, labelKey, Lucide icon, `match(pathname)` active predicate
  - [x] Bottom items: `/`, `/explore`, `/saved`, `/plan` — icons `Home`, `Compass`, `Heart`, `CalendarDays` (UX Phase 2 design.md)
  - [x] Desktop-only items (append after Plan): `/passport`, `/info`; auth control last (not a route match)

- [x] **BottomNav update** (AC: 1, 3, 4)
  - [x] Replace Phase 1 three-tab config (Home · Explore · Info) with four-tab Phase 2 config from `navConfig.ts`
  - [x] Keep existing shell: `fixed inset-x-0 bottom-0`, `h-14`, safe-area padding, accent dot active indicator, `aria-current="page"`
  - [x] Add `className="md:hidden"` on root `<nav>` so bar unmounts visually at ≥768px
  - [x] On link click: `track('nav_destination_tapped', { destination, nav_type: 'bottom' })`

- [x] **DesktopTopNav (new)** (AC: 2, 3, 4)
  - [x] Create `web/src/components/layout/DesktopTopNav.tsx` (client component)
  - [x] Sticky top bar: `hidden md:block`, `h-14`, `bg-surface`, bottom border `border-divider`, `z-50`, safe-area top padding
  - [x] Layout: Powri wordmark/link home (left) · horizontal link row (centre or start) · Sign in button or avatar link to `/account` (right)
  - [x] Reuse `useAuth()` from `AuthProvider` — guest: `openSignIn({ trigger: 'nav' })` on Sign in; authed: avatar image via `resolveAvatarSrc(profile.avatar_url, profile.updated_at)` or initials fallback; link to `/account`
  - [x] Active link: `text-accent` + optional underline or font-semibold — match bottom nav accent semantics
  - [x] On link/button click: `track(..., { nav_type: 'top' })`

- [x] **AppShell responsive layout** (AC: 1, 2)
  - [x] Mount `<DesktopTopNav />` above `<main>`
  - [x] Main padding: keep `pb-[calc(3.5rem+env(safe-area-inset-bottom))]` on mobile only — use `md:pb-0` so desktop content is not offset for hidden bottom nav
  - [x] Do **not** add top padding for DesktopTopNav on pages that already render `DiscoveryTopBar` (home/explore/info) — DesktopTopNav is global; DiscoveryTopBar remains page-local below it on those surfaces. Verify no double-header overlap at md+ (may need `DiscoveryTopBar` top offset or document that desktop shows both global top nav + page search bar — follow UX: global nav row, then page header)

- [x] **Stub routes** (AC: 5)
  - [x] `web/src/app/[locale]/saved/page.tsx` — placeholder heading "Saved resorts" + short copy "Coming soon" or link to Explore
  - [x] `web/src/app/[locale]/plan/page.tsx` — placeholder "Plan your trip" empty-state skeleton (Story 14.2 replaces)
  - [x] `web/src/app/[locale]/passport/page.tsx` — placeholder "Ski Passport" (Story 13.3 replaces)
  - [x] Use `PageContent`, `setRequestLocale`, `getTranslations` patterns from existing `[locale]/explore/page.tsx`

- [x] **i18n** (AC: 1, 2)
  - [x] Add to `web/messages/en.json` → `nav`: `saved`, `plan`, `passport` (Info/home/explore/signIn/account already exist)

- [x] **Analytics registry** (AC: 4)
  - [x] Update `phase2Events.ts` → `nav_destination_tapped.instrumentationFiles`: `['src/components/layout/BottomNav.tsx', 'src/components/layout/DesktopTopNav.tsx']`

- [x] **E2E smoke update** (AC: 1)
  - [x] Replace bottom-nav Info click with desktop viewport **or** hamburger Info link (hamburger still has Info until Story 10.2 refactors menu — acceptable interim)
  - [x] Add mobile 375px four-tab + Saved/Plan stub assertions; desktop 1024px top-nav + Passport stub assertions

- [x] **Unit tests** (AC: 1, 2)
  - [x] `navConfig.test.ts` — route matchers + item list composition
  - [x] `pageInsets.test.ts` — stacked offset constants stay in sync

- [x] **Out of scope — do NOT implement in 10.1**
  - [x] `AppMenuSheet` hamburger dedupe / Passport overflow → **Story 10.2**
  - [x] Save heart, localStorage, `/api/saved` → **Stories 10.3–10.4**
  - [x] Plan tab empty state + sign-in CTA content → **Story 14.2**
  - [x] Passport badge count chip → **Stories 10.2 / 13.x**
  - [x] `nav_type: 'hamburger'` instrumentation → **Story 10.2**

## Dev Notes

### Current state (read before editing)

| File | Today | This story |
|------|-------|------------|
| `BottomNav.tsx` | 3 tabs: Home · Explore · **Info** | 4 tabs: Home · Explore · **Saved · Plan**; `md:hidden` |
| `AppShell.tsx` | Bottom nav only; main always has bottom padding | Add DesktopTopNav; responsive main padding |
| `AppMenuSheet.tsx` | Home · Explore · Info · Account/Sign in | **Unchanged** (10.2) — interim duplication of Home/Explore in hamburger is known; ship 10.2 soon after |
| `DiscoveryTopBar.tsx` | Page-local hamburger + search on home/explore/info | Unchanged; stacks under DesktopTopNav on desktop |

### PO / UX alignment

| Topic | Decision |
|-------|----------|
| Breakpoint | Tailwind `md` = **768px** (FR-16.2, OQ-3 resolved) |
| Bottom nav height | 56px (`h-14`) — unchanged from Phase 1 |
| Desktop top nav height | 56px (`h-14`) per `ux-phase2/design.md` |
| Icons | Lucide: `Home`, `Compass`, `Heart`, `CalendarDays` |
| Active state | Mobile: accent label + 4px dot (preserve Phase 1 pattern). Desktop: accent text |
| Scroll behaviour | Bottom nav never hides on scroll (Phase 1 preserved) |
| Sticky desktop nav | Top nav sticky on scroll |
| Locale routing | `localePrefix: 'never'` — hrefs stay `/explore` not `/en/explore` |

### Desktop + DiscoveryTopBar stacking

At ≥768px, users see **DesktopTopNav** (global) plus page headers (`HomeTopBar`, `ExploreTopBar`, etc.) on discovery surfaces. UX experience.md shows top bar with hamburger/search **below** brand row on mobile; on desktop the global nav replaces bottom tabs but page-level search bars may remain. Implementation approach:

1. DesktopTopNav = full-width sticky strip with Powri + primary links + auth.
2. Discovery pages keep their search/hamburger row **below** DesktopTopNav (two-row header on desktop is acceptable for 10.1).
3. Resort detail / quiz / account pages: only DesktopTopNav + content (no DiscoveryTopBar).

Do not remove DiscoveryTopBar in this story.

### Auth patterns to reuse (Epic 9)

| Pattern | Location |
|---------|----------|
| Auth context | `web/src/components/auth/AuthProvider.tsx` — `isAuthenticated`, `profile`, `openSignIn` |
| Avatar URL | `web/src/lib/auth/avatarUrl.ts` — `resolveAvatarSrc` |
| Sign-in trigger | `openSignIn({ trigger: 'nav' })` — already in `SignInTrigger` union |

### Analytics contract

```ts
track('nav_destination_tapped', {
  destination: 'saved', // slug-like enum, lowercase
  nav_type: 'bottom',   // or 'top'
});
```

Common properties (`is_logged_in`, etc.) merged by `track()` automatically. Do not fire on passive render — only on user tap/click before navigation.

### Regression risks

| Risk | Mitigation |
|------|------------|
| E2E smoke expects Info in bottom nav | Update test in this story |
| `AppMenuSheet` still lists Home/Explore | Documented interim; 10.2 removes dupes |
| Main content hidden behind nav | Verify `pb-*` only on `< md` |
| 404 on new tabs | Stub pages required in 10.1 |

### Cross-story context (Epic 10)

| Story | Depends on 10.1 |
|-------|-----------------|
| **10.2** | Hamburger overflow; removes Home/Explore/Saved/Plan from menu; adds Passport with badge chip |
| **10.3** | Heart on cards/detail — Saved tab must exist first |
| **10.4** | Full `/saved` page replaces stub |

Epic 10 depends on Epic 9.2 (session) — **done**. Auth flows for Sign in / Avatar in desktop nav are ready.

### References

- [Source: epics/phase2/epics-phase2.md — Story 10.1]
- [Source: prds/phase2/shards/12-features-navigation.md — FR-16.1, FR-16.2]
- [Source: ux-designs/ux-phase2/design.md — BottomNav, DesktopTopNav]
- [Source: ux-designs/ux-phase2/experience.md — Navigation model]
- [Source: architecture/phase2/architecture-phase2.md — §6.4 Adaptive navigation]
- [Source: docs/analytics/tracking-plan.md — §3 `nav_destination_tapped`]
- [Source: implementation-artifacts/1-4-app-shell-routing-bottom-navigation.md — Phase 1 BottomNav baseline]

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- E2E smoke failed against stale `localhost:3000` server (reuseExistingServer); passed after fresh build + `CI=1`.

### Completion Notes List

- Phase 2 four-tab bottom nav (Home · Explore · Saved · Plan) with shared `navConfig.ts` and `navAnalytics.ts`.
- New sticky `DesktopTopNav` at md+ with Powri brand, full link row, Sign in / avatar auth control.
- Discovery top bars offset to `md:top-14`; explore/info page padding uses `DISCOVERY_PAGE_TOP_OFFSET_CLASS` for two-row desktop header.
- Stub pages for `/saved`, `/plan`, `/passport` with i18n copy.
- `nav_destination_tapped` wired; `phase2Events.ts` instrumentation files updated.
- E2E: Info reached via desktop top nav after resort detail; quiz intro wait added for animation.
- Follow-up fix: `UnsplashAttribution` links moved outside card `<Link>` wrappers to resolve nested `<a>` hydration errors (visible when navigating after visiting home/explore).
- Review follow-up: `APP_MENU_SHEET_TOP_CLASS` derived from `pageInsets` constants; `AppMenuSheet` uses `APP_MENU_NAV_ITEMS` from `navConfig`; E2E + unit tests for nav matchers and stub routes; smoke test locators updated for dual-nav DOM and post–nested-`<a>` card structure.

### File List

- `web/src/components/layout/navConfig.ts` (new)
- `web/src/components/layout/navConfig.test.ts` (new)
- `web/src/lib/layout/pageInsets.test.ts` (new)
- `web/src/components/layout/navAnalytics.ts` (new)
- `web/src/components/layout/BottomNav.tsx` (modified)
- `web/src/components/layout/DesktopTopNav.tsx` (new)
- `web/src/components/layout/AppShell.tsx` (modified)
- `web/src/components/layout/AppMenuSheet.tsx` (modified)
- `web/src/components/home/HomeTopBar.tsx` (modified)
- `web/src/components/explore/ExploreTopBar.tsx` (modified)
- `web/src/components/info/InfoTopBar.tsx` (modified)
- `web/src/lib/layout/pageInsets.ts` (modified)
- `web/src/app/[locale]/saved/page.tsx` (new)
- `web/src/app/[locale]/plan/page.tsx` (new)
- `web/src/app/[locale]/passport/page.tsx` (new)
- `web/src/app/[locale]/explore/page.tsx` (modified)
- `web/src/app/[locale]/info/page.tsx` (modified)
- `web/messages/en.json` (modified)
- `web/src/lib/analytics/phase2Events.ts` (modified)
- `web/src/components/resort/ResortCard.tsx` (modified — 10.1 follow-up nested `<a>` fix)
- `web/src/components/quiz/QuizTopMatchCard.tsx` (modified — 10.1 follow-up nested `<a>` fix)
- `web/e2e/smoke.spec.ts` (modified)
- `web/playwright.config.ts` (modified — `PLAYWRIGHT_PORT` override for prod-server e2e when dev occupies 3000)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/10-2-hamburger-overflow-menu.md` (new — PO requirements draft for 10.2)

## Change Log

- 2026-07-03: Story 10.1 implemented — Phase 2 adaptive navigation (4-tab mobile bottom nav + desktop top nav), stub routes, analytics, e2e update.
- 2026-07-03: Review follow-up — shared menu sheet offset, navConfig unit tests, adaptive nav E2E coverage.
