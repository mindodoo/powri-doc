# Validation Report — Post-Fix (v1.1)

- **PRD:** `_powri/planning-artifacts/prds/phase1/prd-phase1.md`
- **Run at:** 2026-06-12
- **Grade:** **Good → Ready for epics** — **approved by product owner 2026-06-12**

## Changes applied (per product owner decisions)

| # | Finding | Resolution |
|---|---------|------------|
| 1 | Nav/screen phase mismatch | Phase 1 nav: **Home · Explore · Info**. AI + quiz **Phase 1**. Plan/My Trip tab **Phase 3**. All §12 screens phase-tagged. |
| 2 | 10 vs 20 resorts | **All 20** ship Phase 1; slug column added to §3.1 table. |
| 3 | CMS vs markdown | **§11.7 resolved:** markdown in repo → build-time import; CMS Phase 3 optional. |
| 4 | No FR IDs | **FR-1…FR-7** with acceptance criteria for Phase 1. |
| 5 | Quiz phase mismatch | Quiz promoted to **Phase 1**; tracking plan aligned. |
| 6 | No counter-metrics | Guardrails added to §2; events in tracking plan. |
| Medium | Maps, typography, hero video, terrain map, glossary | OSM default; §12 canonical; video out of scope; link-out trail map; **§18 Glossary**. |
| Medium | Satisfaction collection | Micro-survey spec in §2. |
| Medium | Joetsu Kokusai naming | Fixed in §3.1 table. |

## Remaining watch-items (non-blocking)

1. **Phase 1 scope is ambitious** — all 20 resorts + AI chat + quiz + analytics + legal pages. Content-freeze gate is the real bottleneck.
2. **Explore vs Home** — shared component assumption documented; confirm in architecture if they diverge.
3. **Privacy Policy / ToS** — required before Ring 2 (§15); content not drafted yet.
4. **Unsplash bulk images** — still on hold pending developer app approval.

## Recommended next step

Approve v1.1 → run **CE** (`bmad-create-epics-and-stories`) scoped to Phase 1 FR-1…FR-7.
