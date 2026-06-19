# Story 7.4: Top Bar — Hamburger Menu & Inline Search

Status: done

baseline_commit: b833449

## Story

As a **planner**,  
I want a compact top bar with navigation menu and search,  
So that I can find resorts without redundant titles in the header.

## Acceptance Criteria

1. No “Japan Ski Trip Planner” or “Find your resort” in top bar
2. Explore `h1` “Find your resort” in page body
3. Layout at 375px: **[Hamburger] · [Search]** — inline field on Explore/Info; search icon + expandable panel on Home (PO refinement)
4. Menu: Home, Explore, Info from `nav.*` i18n
5. Placeholder “What are you looking for” in grey
6. Resort name search reuses existing logic
7. Home: transparent over hero when at top; solid when scrolled / menu / search open
8. Explore: always solid top bar
9. Original scope Home + Explore; **Info** also received inline soft search + hamburger (PO extension)

## Tasks / Subtasks

- [x] `DiscoveryTopBar`, `AppMenuSheet`, `ResortSearchField`
- [x] Home (icons + panel), Explore (inline solid), Info (inline soft) top bars
- [x] Explore `h1` in page body; Info page top bar
- [x] i18n + analytics (`nav_menu_opened`, `search_focused`)
- [x] Remove `TopBarWithSearch`, `HomeSearchPanel`
- [x] Post-PO fixes: header X close race, native search clear removed, inline spacing (`pl-top-bar-x gap-top-bar-x pr-screen-x`)
- [x] CR fix: `nav_menu_opened` fires once per open (edge-triggered)
- [x] Validate lint, test:analytics, build

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- `DiscoveryTopBar`: shared shell — `layout="icons"` (Home) or `layout="inline"` (Explore/Info)
- Home: hamburger + search **icon**; panel expands with solid `ResortSearchField`; transparent header over hero until scroll/menu/search
- Explore/Info: hamburger + always-visible inline `ResortSearchField` (`flex-1`); Explore solid tone, Info soft tone (`bg-surface/55`)
- `ResortSearchField`: dropdown on focus; debounced `search_performed`; grey Lucide clear button (`type="text"` + `role="searchbox"`)
- `AppMenuSheet`: scrim + dropdown nav; Escape to close; analytics on open edge only

### File List

- `web/src/components/layout/DiscoveryTopBar.tsx` (new)
- `web/src/components/layout/AppMenuSheet.tsx` (new)
- `web/src/components/search/ResortSearchField.tsx` (new)
- `web/src/components/home/HomeTopBar.tsx`
- `web/src/components/explore/ExploreTopBar.tsx`
- `web/src/components/info/InfoTopBar.tsx` (new — PO extension)
- `web/src/app/[locale]/page.tsx`
- `web/src/app/[locale]/explore/page.tsx`
- `web/src/app/[locale]/info/page.tsx`
- `web/src/styles/tokens.css` (`--space-top-bar-x`)
- `web/messages/en.json`
- `docs/tracking-plan.md`
- `web/src/lib/analytics/phase1Events.ts`
- Deleted: `TopBarWithSearch.tsx`, `HomeSearchPanel.tsx`

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve (with fixes applied)

### AC traceability

| AC | Status |
|----|--------|
| No app title in header | Pass |
| Explore h1 in body | Pass |
| Hamburger + search | Pass — inline Explore/Info; icon+panel Home |
| Nav menu items | Pass |
| Placeholder copy | Pass |
| Search behaviour | Pass — `searchResortsByName`, debounced `search_performed` |
| Home transparent/solid | Pass |
| Explore solid | Pass |
| Scope | Partial — Info top bar added per PO |

### Review Findings

- [x] [Review][Pass] `npm run lint`, `test:analytics`, `npm run build` green
- [x] [Review][Pass] Analytics registry updated for new events
- [x] [Review][Fixed] `nav_menu_opened` double-fire on Home re-render
- [x] [Review][Note] Home search icon pattern is accepted PO deviation from “always-visible field” AC
