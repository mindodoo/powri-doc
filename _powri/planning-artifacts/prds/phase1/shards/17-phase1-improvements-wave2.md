<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §3.9: Phase 1 Improvements Wave 2 (Epic 7)

**Source:** Product owner feedback post-launch · **Epic shard:** [`epics/phase1/shards/epic-07-phase1-ux-wave2.md`](../../../epics/phase1/shards/epic-07-phase1-ux-wave2.md)  
**Timing:** After Epic 6 shipped to production (`main`, 2026-06-17).

---

## Purpose

Epic 6 closed pre-launch polish. Epic 7 captures **post-launch UX feedback** from real device usage — quiz flow, global spacing, navigation chrome, terrain chip readability, and geographically correct “Around resort” copy.

---

## Functional requirements (improvements)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-13** | Quiz questions **slide in-page** (no full-card remount); results **#1 visible** on iPhone SE | 7.1 |
| **IMP-14** | **Reduced top spacing**; responsive **content max-width** and horizontal padding on tablet/desktop | 7.2 |
| **IMP-15** | **Fixed overlay back button** top-left on quiz, detail, legal | 7.3 |
| **IMP-16** | Top bar: **no app title** on Home; **no “Find your resort” in bar**; **hamburger** + **inline search** (“What are you looking for”); Explore **h1 in page** | 7.4 |
| **IMP-17** | Terrain/mountain **stat chips ≤28 chars**; no horizontal chip scroll at 375px | 7.5 |
| **IMP-18** | Daytrips heading uses **`around_area`** (not airport); editorial **resort-area** daytrips | 7.6 |

---

## Analytics changes (Epic 7)

| Event | Trigger | Phase | Story |
|-------|---------|-------|-------|
| `nav_menu_opened` | Hamburger menu opened | 1.2 | 7.4 (optional) |
| `search_focused` | Inline search field focused | 1.2 | 7.4 (optional) |

Update [`docs/analytics/tracking-plan.md`](../../../../../docs/analytics/tracking-plan.md) only if optional events ship.

**No change** to quiz funnel events for 7.1.

---

## Out of scope (Epic 7)

- Bottom nav item changes (still Home · Explore · Info)
- Global search beyond resort name (Phase 2+)
- AI chat, accounts, maps
- Rebranding / new app name in metadata (browser tab title may stay “Japan Ski Trip Planner”)

---

## Release gate

Epic 7 is **recommended complete** before the next manual QA pass. Minimum bar for merge: **7.2 + 7.6** (spacing + around-area bug). **7.1 + 7.4** are highest user-visible UX wins.
