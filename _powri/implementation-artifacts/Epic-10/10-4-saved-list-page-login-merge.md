---
baseline_commit: HEAD
---

# Story 10.4: Saved List Page & Login Merge

Status: done

<!-- PO additions 2026-07-10 — merge rules for shared devices; dismissible guest sync banner; toast→/saved freshness; top-10 expand list (Explore parity) -->
<!-- PO follow-up 2026-07-10 — merge gate: 0 DB rows (Option A); initial visible 10; dismissible banner per session -->

## Story

As a **visitor or signed-in user**,
I want a Saved page listing my hearted resorts and safe sync behaviour on login and logout,
So that my shortlist is easy to review, follows me across devices when appropriate, and never leaks to the next person on a shared device.

**Source shard(s):** [`epics/phase2/shards/epic-10-nav-saved.md`](../../planning-artifacts/epics/phase2/shards/epic-10-nav-saved.md) · [`prds/phase2/shards/11-features-saved-resorts.md`](../../planning-artifacts/prds/phase2/shards/11-features-saved-resorts.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **10.3** (SaveHeart, `SavedResortsProvider`, `/api/saved` CRUD, save toast → `/saved`), Story **9.2** (Supabase session)

**Out of scope:** Saved search/filters (PRD Phase 2 out of scope), sharing saved list, server-side guest reads.

---

## Acceptance Criteria

### A. FR-15.1 — Guest persistence (carry-over from 10.3)

1. **Given** I am a **guest**  
   **When** I save or unsave resorts  
   **Then** slugs persist in `localStorage` key **`powri_saved_resorts`** (deduped array; append on save) on the current device only

2. **And** guest saves survive page refresh on the same browser

### B. FR-15.1 — Authenticated persistence & cross-device sync

3. **Given** I am **signed in**  
   **When** I save or unsave  
   **Then** changes persist via `/api/saved` and sync across devices under the same account (hearts and list reflect DB after refresh on any device)

### C. FR-15.1 — Login merge rules (shared-device safety) — PO 2026-07-10

4. **Given** I am a **guest** with slugs in `localStorage`  
   **When** I sign in and my account has **zero rows** in `saved_resorts` (new sign-up **or** long-existing account that never used Saved)  
   **Then** client calls **`POST /api/saved/merge`** with local slugs  
   **And** server upserts those slugs into `saved_resorts` (dedupe)  
   **And** client **clears** `powri_saved_resorts` from `localStorage` after successful merge  
   **And** `SavedResortsProvider` refreshes from DB so hearts and `/saved` show merged resorts immediately

5. **Given** I am a **guest** with slugs in `localStorage`  
   **When** I sign in and my account **already has ≥1 saved resort** in `saved_resorts`  
   **Then** local guest slugs are **discarded** — not merged, not overwriting account saves  
   **And** `localStorage` key is **cleared**  
   **And** UI reflects **account** saves only (protects account list on shared devices)

6. **And** if merge succeeds with **≥1 slug** added, show a brief success toast: **"Saved resorts synced"** (`saved.mergeToast`, 3s, same `TopToast` pattern as sign-in — no navigation)

7. **And** merge runs once per guest→authenticated transition (guard against duplicate POST on re-renders / strict mode)

### D. FR-15.1 — Logout localStorage hygiene — PO 2026-07-10

8. **Given** I am signed in (or was signed in this session)  
   **When** I **sign out** from any sign-out surface (`AccountSettingsForm`, `AccountDangerZone`, or future auth flows)  
   **Then** **`powri_saved_resorts` is removed** from `localStorage`  
   **And** `SavedResortsProvider` resets to guest-empty state so hearts are **not** shown for the previous user's saves  
   **And** next guest on the device starts with a clean local slate

### E. FR-15.2 — Saved list view (`/saved`)

9. **Given** I open **`/saved`** (public route — no login wall)  
   **When** I have saved resorts (guest local or user synced)  
   **Then** I see **resort cards using the same `ResortCard` component** and list layout as Explore (vertical stack, `gap-card-gap`, same image aspect and metadata)

10. **And** sort order is **most recently saved first** (newest at top, oldest at bottom):
    - **Guest:** reverse of `localStorage` array order (append-on-save → last entry = newest)
    - **User:** `saved_at DESC` from DB (fix `GET /api/saved` ordering — currently ascending in 10.3)

11. **And** list uses **in-page expand** like Explore's `ResortList` **`showMoreMode="expand"`**:
    - Initial render: **top 10** cards — same default as Explore (`EXPLORE_INITIAL_VISIBLE` = 10; omit `initialVisible` override or pass `{10}` explicitly)
    - When more than 10: **Show more** button expands to reveal remaining saved resorts (same button styling/behaviour as Explore expand — not the Home→Explore handoff link)

12. **Given** I have **no** saved resorts  
    **When** I open `/saved`  
    **Then** empty state copy + CTA to Explore (replace stub `placeholderBody`; keep friendly tone)

13. **And** unknown / unpublished slugs (stale localStorage) are **skipped** silently — do not crash the page

### F. FR-15.2 — Toast navigation to `/saved` — PO 2026-07-10

14. **Given** I tap heart to **save** and the save toast appears (**"Saved! View in Saved."**, 5s — Story 10.3)  
    **When** I tap the toast body within the 5s window  
    **Then** I navigate to **`/saved`**  
    **And** the **just-saved resort appears at the top** of the list **without** requiring a manual refresh or artificial delay (provider already holds optimistic slug; list reads live provider state)

15. **And** navigating via bottom/top nav Saved tab shows the same list state as toast navigation

### G. FR-15.2 — Guest sync banner — PO 2026-07-10

16. **Given** I am a **guest** on `/saved`  
    **When** the page renders and I have **not** dismissed the banner this browser session  
    **Then** a banner appears at the **top of the saved page content** (below global nav, above page title/list): **"Sign in to sync saved resorts across devices"** (`saved.syncBanner`)  
    **And** banner includes **Sign in** CTA opening sign-in sheet with `trigger: 'saved'` and `returnTo: '/saved'`  
    **And** styling matches UX **`SyncBanner`**: `--color-highlight-bg` / `sync-banner-bg` (`#EEF3FF`), full-width — see `mockups/key-saved.html`  
    **And** banner is **dismissible per session** via × control (persist dismiss in **`sessionStorage`** — e.g. key `powri_saved_sync_banner_dismissed`; hidden until tab/session ends; reappears on new session)

17. **Given** I am a **guest** and I dismissed the sync banner  
    **When** I remain in the same browser session  
    **Then** the banner stays hidden on subsequent `/saved` visits

18. **Given** I am **signed in**  
    **When** I view `/saved`  
    **Then** the sync banner is **hidden**

### H. Analytics — FR-15 / tracking plan

19. **And** `saved_list_viewed` fires once per `/saved` page visit with `saved_count`, `is_logged_in`

20. **And** `phase2Events.ts` instrumentation path updated for `saved_list_viewed`

### I. Accessibility & regression

21. **And** sync banner Sign-in and dismiss (×) controls have accessible names; banner uses `role="region"` + `aria-label` where helpful

22. **And** saved list is a semantic list (`<ul>`) like Explore; heart buttons on cards still work (unsave updates list)

23. **And** no regression to Story 10.3 heart states, toasts, or nav `/saved` tab routing

---

## PO decisions (resolved 2026-07-10)

| Topic | Decision |
|-------|----------|
| Merge when (**Option A**) | Account has **0** DB saved rows → merge local slugs (includes long-existing accounts who never saved); **≥1** DB rows → discard local |
| Logout | Always **clear** `powri_saved_resorts` |
| List initial count | **10** + in-page **Show more** — same as Explore (`EXPLORE_INITIAL_VISIBLE`, `showMoreMode="expand"`) |
| Card template | Reuse **`ResortCard`** via **`ResortList`** (same as Explore) |
| Sort | Newest saved **first** |
| Sync banner | **Dismissible per session** (× + `sessionStorage`); guest-only; aligns with PRD §5.8 / UX D11 |
| Merge feedback | Toast **"Saved resorts synced"** when ≥1 slug merged (UX experience.md) |
| Toast → `/saved` | Must show **immediate** list including just-saved resort (no loading gate if provider ready) |

---

## User journey

```mermaid
flowchart TD
  subgraph saveFlow [Save → Saved]
    A[Tap heart on card/detail] --> B[Provider optimistic save]
    B --> C[Toast: Saved! View in Saved. 5s]
    C --> D{Tap toast?}
    D -->|Yes| E[/saved — newest card at top]
    D -->|No| F[Auto-dismiss]
  end

  subgraph guestSaved [Guest /saved]
    G[Open Saved tab] --> H[SyncBanner + resort cards]
    H --> I{Has saves?}
    I -->|No| J[Empty state → Explore CTA]
    I -->|Yes| K[Top 10 + Show more]
  end

  subgraph loginMerge [Login merge]
    L[Guest with local slugs signs in] --> M{DB saved count?}
    M -->|0 rows| N[POST /api/saved/merge]
    N --> O[Clear localStorage + merge toast]
    M -->|≥1 rows| P[Discard local + clear localStorage]
    N --> Q[Hearts + list from account]
    P --> Q
  end

  subgraph logout [Logout]
    R[Sign out] --> S[Clear localStorage]
    S --> T[Provider guest-empty — no stale hearts]
  end
```

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `clearSavedSlugs` / merge client helper; `POST /api/saved/merge` route (0-row merge, discard when ≥1 row, dedupe, auth 401); slug→list ordering helper
- [ ] **Quiz / scoring:** N/A
- [x] **Analytics:** `saved_list_viewed` in tracking plan if needed + `phase2Events.ts` + `npm run test:analytics`
- [ ] **Content / resorts:** N/A
- [x] **User-facing flow:** `e2e/saved-list-guest.spec.ts` — guest saves → `/saved` newest-first; toast tap + nav tab parity; sync banner dismiss per session; show-more expand (11 slugs); logout hygiene via localStorage clear; unsave on list (8/8 pass)
- [x] **Manual QA:** 375px + 1024px — banner dismissible per session for guest; top-10 expand; merge when 0 DB rows (incl. old account); discard local when account has saves; sign-out clean slate

**Pre-merge from `web/`:** `npm run lint && npm run build && npm run test:unit && npm run test:analytics` (+ E2E if spec added)

---

## Tasks / Subtasks

- [x] **Lib — localStorage clear** (AC: 8)
  - [x] Add `clearSavedSlugs()` to `web/src/lib/saved/localStorage.ts` + unit tests
- [x] **API — merge endpoint** (AC: 4–5, 7)
  - [x] `web/src/app/api/saved/merge/route.ts` — POST `{ slugs: string[] }`; if user has ≥1 existing row → `{ merged_count: 0, discarded: true }`; else upsert deduped slugs
  - [x] Route tests (401, merge fresh account, discard when existing saves, idempotent)
- [x] **API — GET sort fix** (AC: 10)
  - [x] `GET /api/saved` → order `saved_at` **descending**; update route test expectations
- [x] **Provider — merge + logout** (AC: 4–8, 14)
  - [x] On `isAuthenticated` transition guest→user: read local slugs → call merge or discard per DB gate → clear local → refresh slugs
  - [x] On sign-out (subscribe auth event or exported hook): `clearSavedSlugs()` + reset state
  - [x] Expose **`savedSlugsOrdered: string[]`** (or equivalent) for list page sort source of truth
  - [x] Merge success toast via `TopToast`
- [x] **Sign-out wiring** (AC: 8)
  - [x] Ensure all `signOut()` paths trigger saved cleanup (Account settings, danger zone, central auth helper if extracted)
- [x] **UI — SyncBanner** (AC: 16–18, 21)
  - [x] `web/src/components/saved/SavedSyncBanner.tsx` — guest-only; `openSignIn({ trigger: 'saved', returnTo: '/saved' })`; × dismiss persisted in `sessionStorage` for current session
- [x] **UI — Saved list** (AC: 9–13, 22)
  - [x] `web/src/components/saved/SavedResortsList.tsx` — map ordered slugs → `ResortListItem[]`; `ResortList` with `listContext="saved"`, `showMoreMode="expand"` (default **10** visible — match Explore; no `initialVisible` override unless explicit `{10}`)
  - [x] Replace stub in `web/src/app/[locale]/saved/page.tsx` — RSC shell + client list + banner
- [x] **Analytics** (AC: 19–20)
  - [x] Fire `saved_list_viewed` on `/saved` mount (client)
  - [x] Update `phase2Events.ts` instrumentation paths
- [x] **i18n** (AC: 6, 12, 16–17)
  - [x] `saved.syncBanner`, `saved.syncBannerSignIn`, `saved.syncBannerDismiss`, `saved.mergeToast`, empty-state copy (replace `placeholderBody`)

---

## Dev Notes

### Reuse — do not reinvent

| Pattern | Location | Use for |
|---------|----------|---------|
| Resort cards list | `ResortList.tsx` + `ResortCard.tsx` | Saved list rendering |
| Initial visible = 10 | `EXPLORE_INITIAL_VISIBLE` in `ResortList.tsx` | Saved page — `showMoreMode="expand"` without override (Explore parity) |
| Save toast → `/saved` | `SavedResortsProvider.tsx` | Already navigates; ensure list consumes same provider slugs |
| Top toast | `TopToast.tsx` | Merge toast (3s) |
| Sign-in from CTA | `AuthProvider.openSignIn` | Sync banner; trigger `'saved'` already in allowlist |
| Guest storage | `localStorage.ts` | Extend with `clearSavedSlugs` |
| Content lookup | `getAllResorts()` / `getResortBySlug()` | Map slugs → `ResortListItem` via `toResortListItem` |

### Current file state (read before editing)

**`saved/page.tsx`** — Stub only: `PageHeader` + `placeholderBody` + Explore link. Replace body with `SavedSyncBanner` + `SavedResortsList`; keep `PageHeader` title.

**`SavedResortsProvider.tsx`** — Loads guest local or GET `/api/saved` on auth change; **does not** merge or clear on logout yet. Slugs order for guests matches localStorage append order (oldest→newest); **reverse for display**.

**`GET /api/saved`** — Returns slugs ordered **`saved_at` ascending** (oldest first) — **invert to descending** for provider + list.

**`localStorage.ts`** — No `clearSavedSlugs()` yet.

**`/api/saved/merge`** — **Does not exist** — create in this story.

### Merge algorithm (canonical)

```
onAuthTransition(guest → authenticated):
  localSlugs = getSavedSlugs()
  if localSlugs.length === 0:
    refresh from GET /api/saved
    return

  response = POST /api/saved/merge { slugs: localSlugs }
  clearSavedSlugs()  // always after login attempt, success or discarded

  if response.merged_count > 0:
    show merge toast

  refresh provider slugs from GET /api/saved
```

**Server `POST /api/saved/merge`:**

```
requireUser()
existing = SELECT count(*) FROM saved_resorts WHERE user_id = ?
if existing >= 1:
  return { merged_count: 0, discarded: true }

for slug in dedupe(body.slugs):
  upsert saved_resorts (ignore duplicates)
return { merged_count: N, discarded: false }
```

Validate slugs are non-empty strings; optionally filter to known resort slugs server-side (client also filters for display).

### Logout algorithm

```
onAuthTransition(authenticated → guest) OR explicit signOut:
  clearSavedSlugs()
  setSlugs([]) or reload guest empty
```

Wire via `supabase.auth.onAuthStateChange` `SIGNED_OUT` in `SavedResortsProvider` **and** ensure sign-out buttons don't leave stale data if event ordering differs.

### Saved list data flow

1. **Provider** exposes ordered slug array (guest: reversed localStorage; user: API desc).
2. **SavedResortsList** (client): `useMemo` map slugs → `ResortListItem[]` preserving order; filter missing resorts.
3. Pass to **`ResortList`** — do **not** re-sort alphabetically.

### Toast → `/saved` freshness (AC 14)

No extra fetch delay: list component reads **`useSavedResorts()`** slug set populated optimistically on save **before** toast shows. If `isReady` is false briefly, show skeleton or prior list — but do not empty list after successful save navigation.

### SyncBanner placement

```
AppShell nav
└─ PageContent (/saved)
   ├─ SavedSyncBanner (guest only, dismissible per session)  ← top of page content
   ├─ PageHeader ("Saved resorts")
   └─ SavedResortsList
```

Banner sits **inside** page content, not competing with `TopToast` (z-index 70).

### Cross-story boundaries

| Story | Owns |
|-------|------|
| **10.3 (done)** | Heart UI, CRUD `/api/saved`, save/unsave toasts, guest localStorage write |
| **10.4 (this)** | `/saved` list UI, sync banner, merge API + login/logout hygiene, `saved_list_viewed` |

### Previous story intelligence (10.3)

- `SavedResortsProvider` inside `AuthProvider` in root layout — extend, don't duplicate
- E2E: fresh dev server (`10-2-2` lesson)
- Auth E2E for signed-in save was deferred — cover merge + authed list here if feasible
- GET failure for authed user still leaves empty hearts — acceptable; list page may show empty with optional error pattern from 10.3 advisory

### Project Structure Notes

- New: `web/src/app/api/saved/merge/route.ts`, `route.test.ts`
- New: `web/src/components/saved/SavedSyncBanner.tsx`, `SavedResortsList.tsx`
- Update: `web/src/app/[locale]/saved/page.tsx`, `SavedResortsProvider.tsx`, `localStorage.ts`, `web/src/app/api/saved/route.ts` (sort)
- Update: sign-out call sites if cleanup not fully handled in provider
- Update: `web/messages/en.json`, `phase2Events.ts`, analytics helper

### References

- [Source: `_powri/planning-artifacts/prds/phase2/shards/11-features-saved-resorts.md` — FR-15.1, FR-15.2]
- [Source: `_powri/planning-artifacts/epics/phase2/epics-phase2.md` — Story 10.4 AC]
- [Source: `_powri/planning-artifacts/architecture/phase2/architecture-phase2.md` — §4.4 post-login merge, §5 `/api/saved/merge`, §6.1–6.3]
- [Source: `_powri/planning-artifacts/ux-designs/ux-phase2/design.md` — SyncBanner]
- [Source: `_powri/planning-artifacts/ux-designs/ux-phase2/mockups/key-saved.html`]
- [Source: `_powri/planning-artifacts/ux-designs/ux-phase2/experience.md` — merge toast, guest saved flow]
- [Source: `_powri/implementation-artifacts/Epic-10/10-3-save-resort-heart-guest-auth.md` — provider, toast, localStorage shape]
- [Source: `docs/analytics/tracking-plan.md` — `saved_list_viewed`]
- [Source: `web/src/components/resort/ResortList.tsx` — expand / initial visible constants]

---

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

### Completion Notes List

- Implemented `clearSavedSlugs`, `orderSavedSlugsNewestFirst`, `mapOrderedSlugsToResortItems` with unit tests.
- Added `POST /api/saved/merge` — rejects unknown slugs (400), merges when 0 DB rows, discards when ≥1 row; module-level promise dedupes duplicate POST on login.
- Fixed `GET /api/saved` to `saved_at DESC`; auth saves prepend optimistically so toast→`/saved` shows newest first (AC 14).
- `SavedResortsProvider`: guest→auth merge + clear localStorage; logout via auth transition + `SIGNED_OUT` listener; merge toast 3s.
- `/saved` UI: `SavedSyncBanner` (one-row layout, session dismiss), `SavedResortsList` via `ResortList` expand/top-10, title-only header, empty state copy per PO decision A.
- `saved_list_viewed` fires once per visit when provider ready; phase2Events paths updated.
- E2E: `e2e/saved-list-guest.spec.ts` (8 tests) — guest list, banner, toast→/saved, show-more, logout hygiene.
- Code review follow-ups: filter stale slugs before merge; GET retry; login transition keeps hearts visible (no slug/`isReady` reset on guest→auth).
- Verified: `npm run lint && npm run test:unit && npm run test:e2e -- e2e/saved-list-guest.spec.ts e2e/save-heart-guest.spec.ts` — 193 unit + 13 E2E pass.

### File List

- `web/src/lib/saved/localStorage.ts` (modified)
- `web/src/lib/saved/localStorage.test.ts` (modified)
- `web/src/lib/saved/orderSlugs.ts` (new)
- `web/src/lib/saved/orderSlugs.test.ts` (new)
- `web/src/app/api/saved/route.ts` (modified)
- `web/src/app/api/saved/route.test.ts` (modified)
- `web/src/app/api/saved/merge/route.ts` (new)
- `web/src/app/api/saved/merge/route.test.ts` (new)
- `web/src/components/saved/SavedResortsProvider.tsx` (modified)
- `web/src/components/saved/SavedSyncBanner.tsx` (new)
- `web/src/components/saved/SavedResortsList.tsx` (new)
- `web/src/components/saved/SavedPageContent.tsx` (new)
- `web/src/app/[locale]/saved/page.tsx` (modified)
- `web/src/components/resort/ResortList.tsx` (modified)
- `web/src/lib/analytics/savedAnalytics.ts` (modified)
- `web/src/lib/analytics/phase2Events.ts` (modified)
- `web/messages/en.json` (modified)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `web/e2e/saved-list-guest.spec.ts` (new)

### Review Findings

- [x] [Review][Decision] Merge failure still clears localStorage — confirmed intentional per PO algorithm; no retry on failure.
- [x] [Review][Patch] Filter unknown slugs before merge POST [`SavedResortsProvider.tsx`] — `filterPublishedSavedSlugs` applied before POST.
- [x] [Review][Patch] Auth GET failure leaves empty hearts [`SavedResortsProvider.tsx`] — 3-attempt retry; login transition keeps slugs until fetch succeeds.
- [x] [Review][Patch] Login transition empty-hearts flash [`SavedResortsProvider.tsx`] — skip slug clear + `isReady` reset on guest→auth transition.
- [x] [Review][Defer] `saved_count` analytics may exceed visible cards [`SavedPageContent.tsx:29`] — deferred, stale slugs inflate count while list filters silently.
- [x] [Review][Defer] Non-atomic merge count gate [`merge/route.ts:46`] — deferred, TOCTOU edge case on concurrent tab login.
- [x] [Review][Defer] Orphaned localStorage on session-restore without merge [`SavedResortsProvider.tsx:168`] — deferred, requires `previousAuthenticated === false` which is null on cold load with existing session.

### Change Log

- 2026-07-10: Story created with PO merge/shared-device rules, guest sync banner, top-10 expand list, toast→/saved freshness (create-story 10.4).
- 2026-07-10: PO follow-up — merge Option A (0 DB rows); initial visible 10 (Explore parity); dismissible sync banner per session.
- 2026-07-10: Implementation complete — saved list, merge API, login/logout hygiene, analytics; E2E deferred pending manual QA.
- 2026-07-10: Code review + E2E (`saved-list-guest.spec.ts`, 8/8 pass). 1 decision-needed, 2 patch, 4 defer findings logged.
- 2026-07-10: Review follow-ups applied — merge slug filter, GET retry, login flash fix; story marked done.
