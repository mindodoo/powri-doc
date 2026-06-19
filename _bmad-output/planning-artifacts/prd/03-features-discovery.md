# PRD §3.1: Resort Discovery (Home) · FR-1

**Source:** [`../prd.md`](../prd.md) §3.1 · **Epic:** [`../epics/epic-02-discover.md`](../epics/epic-02-discover.md) Stories 2.2–2.6 · **Screens:** [`13-ux-screens.md`](13-ux-screens.md#screen-1-home)

---

## Purpose

Present all **20** Japan ski destinations in a browsable, visually stunning format. First impression = premium travel magazine.

## Requirements

- Resort cards: hero photo, name, region, 1-line tagline
- Filter/sort: Region, Skill Level, Snow Quality, Distance from Airport
- Search bar for resort name
- Badge summary on cards (e.g. 🏔 Varied terrain · ⭐ Powder heaven)
- Card tap → Resort Detail
- Lazy-load images; WebP

## Explore tab

Same 20-resort dataset and components as Home — **not a separate data source**. Explore foregrounds filter chips + quiz CTA (no hero carousel). See PRD §12.5 Screen 1 vs epic Story 2.6.

## The 20 destinations (Phase 1)

All ship at launch. Slugs match `content/resorts/<slug>.md`.

| # | Resort | Slug | Prefecture | Gateway |
|---|--------|------|------------|---------|
| 1 | Niseko United | `niseko-united` | Hokkaido | CTS |
| 2 | Furano | `furano` | Hokkaido | CTS |
| 3 | Rusutsu | `rusutsu` | Hokkaido | CTS |
| 4 | Kiroro | `kiroro` | Hokkaido | CTS |
| 5 | Sahoro | `sahoro` | Hokkaido | CTS |
| 6 | Zao Onsen | `zao-onsen` | Yamagata | SDJ |
| 7 | Appi Kogen | `appi-kogen` | Iwate | HNA |
| 8 | Nekoma (Alts Bandai) | `nekoma-alts-bandai` | Fukushima | SDJ |
| 9 | Hakuba Valley | `hakuba-valley` | Nagano | MMJ/NRT |
| 10 | Shiga Kogen | `shiga-kogen` | Nagano | NRT |
| 11 | Nozawa Onsen | `nozawa-onsen` | Nagano | NRT |
| 12 | Myoko Kogen | `myoko-kogen` | Niigata | NRT |
| 13 | Naeba | `naeba` | Niigata | NRT |
| 14 | GALA Yuzawa | `gala-yuzawa` | Niigata | Shinkansen |
| 15 | Madarao Kogen | `madarao-kogen` | Nagano | NRT |
| 16 | Joetsu Kokusai | `joetsu-kokusai` | Niigata | NRT |
| 17 | Tomamu | `tomamu` | Hokkaido | CTS |
| 18 | Lotte Arai | `lotte-arai` | Niigata | Shinkansen |
| 19 | Tsugaike Kogen | `tsugaike-kogen` | Nagano | MMJ |
| 20 | Kandatsu Kogen | `kandatsu-kogen` | Niigata | Shinkansen |

**Full table with distances:** [`../prd.md`](../prd.md) lines 127–148

## Analytics

`resort_card_tapped`, `filter_applied`, `filter_zero_results`, `search_performed` — see [`../../../docs/tracking-plan.md`](../../../docs/tracking-plan.md)
