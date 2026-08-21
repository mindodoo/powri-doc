---
baseline_commit: 92d3c45b46a8e497a5c63ba8a8ddad1004812a3b
---

# Story UX-9: Mobile Explore search icon (desktop unchanged)

Status: done

Version when done: **2.12.2.9**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a mobile user on Explore,  
I want a search icon on the right like Home,  
so that the top bar is not an always-open search field.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-9, D7).

## Acceptance Criteria

1. **Mobile Explore:** no always-visible `ResortSearchField` in the top bar. **Search icon on the right**; tap opens the **same panel pattern as Home** (`DiscoveryTopBar` `layout="icons"` + `ResortSearchField` when open). Reuse — do not rebuild search.
2. **Mobile Account:** no search box (today `MobileOverflowNav` has hamburger only — **do not add** a search field). Verify Back removal is UX-7, not this story.
3. **Desktop:** **unchanged** — `DesktopTopNav` already uses a persistent search **icon** that opens the field. Do not alter desktop search.
4. **Mobile Info/About** inline search is **out of scope** unless required later by UX-10 logo centering.
5. `search_focused` / existing search analytics still fire from `ResortSearchField`.

## Testing & Definition of Done

- [x] **Unit:** N/A
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** No new events; `search_focused` if already on field focus
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** Manual Explore 375px: icon → panel → results. Desktop Explore: icon as today. `e2e/smoke.spec.ts` if it types into Explore header field
- [x] **Manual QA:** PO — mobile only change

## Tasks / Subtasks

- [x] `ExploreTopBar`: `layout="icons"` (AC: 1)
- [x] Pass `listContext="explore"` into the open panel (Home currently hardcodes `listContext="home"` on the dropdown field — **reuse component**, set context correctly so analytics `list_context` stays explore) (AC: 1)
- [x] Confirm Account has no inline search (AC: 2)
- [x] Do not edit `DesktopTopNav` search (AC: 3)

## Dev Notes

**Today:** `ExploreTopBar` / `InfoTopBar` use `layout="inline"` (hamburger + field). `HomeTopBar` uses `layout="icons"`. `DiscoveryTopBar` is `md:hidden` — desktop is a different component. When `layout="icons"` and search opens, the field is forced `listContext="home"` — **fix that** so Explore can pass `listContext="explore"`.

**Do not:** Duplicate `ResortSearchField`. Change desktop. Add search to Account.

### Files

| Path | Role |
|------|------|
| `web/src/components/explore/ExploreTopBar.tsx` | `layout="icons"` |
| `web/src/components/layout/DiscoveryTopBar.tsx` | Allow icons layout to use `listContext` prop on the panel |
| `web/src/components/home/HomeTopBar.tsx` | Preserve |
| `web/src/components/search/ResortSearchField.tsx` | Reuse |

### References

- PRD UX-9, D7, §3.2 Explore
- Story 7.4 hamburger + search (Explore was inline)

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 1 pre-existing warning (`supabaseStorage.ts` unused `contentType`)
- `npm run build`: pass
- `npm run test:launch`: pre-existing fail (`NEXT_PUBLIC_CONTACT_EMAIL` not in this shell); not caused by this change
- Playwright `e2e/smoke.spec.ts`: 23 passed, including UX-9 Explore icon → panel → results, desktop icon, Account no search
- Full `npm run test:e2e`: 43 passed; 3 pre-existing fails (`account-avatar-upload` Supabase user create; `review-text-crud` leftover E2E reviews / strict-mode)

### Completion Notes List

- 2026-08-21: Switched `ExploreTopBar` to `DiscoveryTopBar` `layout="icons"`. Icons-panel `ResortSearchField` now uses the `listContext` prop (Explore = `explore`, Home still `home`). `DesktopTopNav` and `InfoTopBar` untouched. Account overflow nav still hamburger-only (smoke assertion). No new analytics events.

### File List

- `_powri/implementation-artifacts/UX/ux-9-mobile-explore-search-icon.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/components/explore/ExploreTopBar.tsx`
- `web/src/components/layout/DiscoveryTopBar.tsx`
- `web/e2e/smoke.spec.ts`

### Change Log

- 2026-08-21: Mobile Explore search matches Home (icon → panel). Panel analytics surface follows `listContext`. Desktop search unchanged.
- 2026-08-21: Code review complete; story marked done.

### Implementation Plan

Reuse `DiscoveryTopBar` icons layout on Explore. Stop hardcoding `listContext="home"` on the open-panel field so Explore analytics stay `explore`. Guard with smoke e2e; do not touch desktop nav or Account chrome.

### Review Findings

- [x] [Review][Defer] UX-9 e2e does not assert `search_focused` with `surface=explore` — optional; `ResortSearchField` receives `listContext="explore"` by inspection [`web/src/components/layout/DiscoveryTopBar.tsx:128`]
- [x] [Review][Defer] Explore search-panel close (X toggle) not e2e-automated — mirrors Home; open + results assertion sufficient for merge [`web/e2e/smoke.spec.ts:312-327`]

### Review Findings (summary)

**Outcome: Approve** — AC 1–5 met. Explore switches to icons layout; panel `listContext` follows prop (fixes prior hardcoded `home`). Desktop, Account, and Info untouched. Smoke covers icon → panel → results, desktop icon-only chrome, Account no search.

**Dismissed (noise / intentional):** Stale `searchTone` JSDoc still mentions explore inline styling — cosmetic only; Info remains the sole inline consumer. Menu + search simultaneously open — pre-existing Home behavior, not introduced here.

### Senior Developer Review (AI)

**Review date:** 2026-08-21  
**Outcome:** Approve

#### Action Items

- [x] [Review][Defer] Analytics surface not e2e-verified — wiring correct; no new events required by AC5
- [x] [Review][Defer] Close-search interaction not e2e-automated — optional hardening
