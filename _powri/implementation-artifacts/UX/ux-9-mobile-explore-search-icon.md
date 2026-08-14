# Story UX-9: Mobile Explore search icon (desktop unchanged)

Status: ready-for-dev

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

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** No new events; `search_focused` if already on field focus
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual Explore 375px: icon → panel → results. Desktop Explore: icon as today. `e2e/smoke.spec.ts` if it types into Explore header field
- [ ] **Manual QA:** PO — mobile only change

## Tasks / Subtasks

- [ ] `ExploreTopBar`: `layout="icons"` (AC: 1)
- [ ] Pass `listContext="explore"` into the open panel (Home currently hardcodes `listContext="home"` on the dropdown field — **reuse component**, set context correctly so analytics `list_context` stays explore) (AC: 1)
- [ ] Confirm Account has no inline search (AC: 2)
- [ ] Do not edit `DesktopTopNav` search (AC: 3)

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

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
