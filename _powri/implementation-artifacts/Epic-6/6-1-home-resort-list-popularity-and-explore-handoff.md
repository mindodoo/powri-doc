# Story 6.1: Home Resort List — Popularity Sort & Explore Handoff

Status: done

baseline_commit: 8f3fa8d

## Story

As a **first-time visitor**,  
I want the Home page to show the most popular resorts first and a short list,  
So that I see the best-known destinations immediately and browse the full catalog on Explore.

## Acceptance Criteria

1. **Given** the Home discovery section with filter "All"  
   **When** the page loads  
   **Then** resorts are ordered by **`popularity_rank` ascending** (1 = most popular), not alphabetically

2. **And** exactly **5** resort cards render by default on Home

3. **And** **"Show all resorts →"** appears when total published > 5

4. **When** I tap that control  
   **Then** I navigate to **`/explore`** (no in-page expansion on Home)

5. **And** Explore retains in-page **Show more** (10 initial) behaviour

6. **And** **`home_explore_cta_clicked`** fires with `source`, `visible_count`, `total_count`

## Tasks / Subtasks

- [x] **Content schema** — `popularity_rank` on all 20 resorts + `_TEMPLATE.md`
- [x] **Sort** — `sortResortsByPopularity.ts`; applied in `getAllResorts()`
- [x] **Home list UX** — `ResortList` `showMoreMode: explore`, `HOME_INITIAL_VISIBLE = 5`
- [x] **DiscoveryList** — home + filter `all` → explore handoff; filtered → expand
- [x] **Analytics** — `home_explore_cta_clicked` in registry + `tracking-plan.md`
- [x] **Launch gate** — `popularity_rank` uniqueness check in `verify-launch-gates.ts`
- [x] **Validate** — `test:launch`, `test:analytics`, lint, build

## Dev Agent Record

### Agent Model Used

Composer

### Default popularity order (editorial)

| Rank | Resort |
|------|--------|
| 1 | Niseko United |
| 2 | Hakuba Valley |
| 3 | Rusutsu |
| 4 | Nozawa Onsen |
| 5 | Furano |
| 6 | Kiroro |
| 7 | Shiga Kogen |
| 8 | Myoko Kogen |
| 9 | Appi Kogen |
| 10 | Zao Onsen |
| 11 | GALA Yuzawa |
| 12 | Tomamu |
| 13 | Lotte Arai |
| 14 | Madarao Kogen |
| 15 | Naeba |
| 16 | Tsugaike Kogen |
| 17 | Joetsu Kokusai |
| 18 | Kandatsu Kogen |
| 19 | Nekoma ALTs Bandai |
| 20 | Sahoro |

### Completion Notes

- Home filtered lists still use in-page expand (only unfiltered Home uses Explore handoff)
- `DEFAULT_POPULARITY_RANK_BY_SLUG` fallback if frontmatter rank missing at parse time

### File List

- `content/resorts/*.md` (20 files — `popularity_rank`)
- `content/resorts/_TEMPLATE.md`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/types.ts`
- `web/src/lib/content/loadResorts.ts`
- `web/src/lib/content/sortResortsByPopularity.ts` (new)
- `web/src/components/resort/ResortList.tsx`
- `web/src/components/resort/DiscoveryList.tsx`
- `web/src/lib/analytics/phase1Events.ts`
- `web/scripts/verify-launch-gates.ts`
- `web/messages/en.json`
- `docs/analytics/tracking-plan.md`

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

All six ACs satisfied. Popularity sort applies globally via `getAllResorts()` so Home and Explore share consistent editorial order. Home handoff cleanly separates discovery depth (Explore) from landing page brevity (5 cards).

### Review Findings

- [x] [Review][Pass] 20 unique `popularity_rank` values; launch gate enforces
- [x] [Review][Pass] Home shows 5 + Explore link; Explore keeps expand at 10
- [x] [Review][Pass] `home_explore_cta_clicked` in registry + tracking-plan §2.1
- [x] [Review][Pass] `test:launch`, `test:analytics`, lint, build green
- [x] [Review][Note] Filtered Home still expands in-page — intentional per filtered UX
- [x] [Review][Note] Re-order ranks by editing `popularity_rank` in resort markdown
