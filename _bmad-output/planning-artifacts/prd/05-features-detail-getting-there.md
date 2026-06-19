# PRD §3.2 & §3.6: Resort Detail (FR-2) & Getting There (FR-3)

**Source:** [`../prd.md`](../prd.md) §3.2, §3.6 · **Epic:** [`../epics/epic-03-detail.md`](../epics/epic-03-detail.md) · **Screens:** [`13-ux-screens.md`](13-ux-screens.md)

---

## FR-2 — Resort Detail Page

### Sections

**Hero & overview:** Static hero (no video V1), name, prefecture, mountain stats, season-labelled dates, snow quality badge.

**Highlights & lowlights:** Two styled sections with icons. Curated per resort — honest lowlights required (see CONTENT_MODEL).

**Terrain map (Phase 1):** Link/button to official `trail_map_url` — no embedded map V1.

**Resort data card:** Skiable area, runs, longest run, lifts, lift pass, terrain %, snowfall, best months, nearest town/village/airport.

**Attribution:** Unsplash on every hero (§17.2, UX-DR14).

---

## FR-3 — Getting There

### 6a. Gateway airport guide
Airport overview + top 3–5 city highlights + pre/post stay options.

### 6b. Transportation to resort
Car, shuttle, train, Shinkansen+bus — cost, duration, recommended option, booking links (clean outbound, §11.1).

### 6c. Season planning
Open season, peak vs off-peak, trip length, lift hours.

### 6d. Practical tips
Ski pass, IC card, cash/card, onsen etiquette, language, weather apps, insurance.

**≥2 transport options per resort** with duration/cost + external links.

---

## Components (Phase 1)

`QuickVerdict`, `HighlightLowlightRow`, `StatRow`, `TransportRow`. Floating "Plan with AI →" CTA (FR-6, Story 3.6) is **Phase 3** — see [`../epics/phase-3-monetisation-ai.md`](../epics/phase-3-monetisation-ai.md).

**Content schema:** [`../../../content/CONTENT_MODEL.md`](../../../content/CONTENT_MODEL.md)
