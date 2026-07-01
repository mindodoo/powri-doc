# Product Requirements Document
## Japan Ski & Snowboard Trip Planner — Mobile Web Application

**Version:** 1.1  
**Status:** Ready for epics (approved 2026-06-12)  
**Last Updated:** June 2026  

> **AI agents:** Do not load this entire file (~1,465 lines) unless you need full § detail.  
> **Start at [`INDEX.md`](../../INDEX.md)** — task lookup + sharded PRD in [`prds/phase1/shards/`](shards/) (~50–150 lines each).  
> Example: quiz work → [`features/quiz/brief.md`](../../features/quiz/brief.md) only.

---

## 1. Product Overview

### 1.1 Vision
A mobile-first web application that helps ski and snowboard enthusiasts plan complete Japan snow trips — from choosing the right resort and finding accommodation, to exploring nearby villages, towns, and the arrival city at their gateway airport.

### 1.2 Problem Statement
Planning a ski trip to Japan is complex. Travellers must research resorts independently, cross-reference accommodation options, figure out transportation to remote locations, find good food, and understand local culture — all across disparate sources in a foreign language. There is no single product that brings this together in a beautiful, trustworthy, up-to-date experience.

### 1.3 Target Users
- International and domestic skiers/snowboarders visiting Japan
- Beginner to advanced skill levels
- Solo travellers, couples, families, and groups
- People who value both on-mountain experience AND cultural exploration

### 1.4 Design Philosophy
Minimal white aesthetic with high-quality photography of resorts and local food. Clean typography, generous whitespace, and intuitive mobile navigation. The product should feel like a premium travel magazine, not a booking engine.

---

## 2. Goals & Success Metrics

| Goal | Metric |
|------|--------|
| Help users discover the right resort | ≥ 70% of users engage with resort detail pages |
| Drive accommodation decisions | ≥ 40% of users interact with the accommodation map *(Phase 2 metric — track from Phase 2 launch)* |
| Improve trip planning confidence | User satisfaction score ≥ 4.2/5 |
| Keep information current | Data freshness: resort info updated at least once per season |

**Counter-metrics (guardrails — do not regress while optimising north-star metrics):**

| Guardrail | Target / watch | Why |
|-----------|----------------|-----|
| Home bounce rate | ≤ 50% single-page sessions without a detail view | Catches discovery UX failures masked by high detail-page % among those who stay |
| Filter-zero-results rate | ≤ 10% of filter applications return empty lists | Catches bad filter logic or content gaps |
| External-link CTR (accommodation / official / trail map) | Baseline at launch; monitor trend | Proxy for real planning intent; sudden drop signals broken links or trust issues |

**Satisfaction collection (≥ 4.2/5 goal):** lightweight in-app micro-survey — show after the user views **2nd distinct resort detail page** in a session (once per session max). Single question: *"How helpful is this for planning your trip?"* (1–5 stars). Instrument via PostHog event `satisfaction_prompt_submitted` (§13). Defer the metric to Phase 2 if Phase 1 traffic is too low for meaningful samples.

---

## 3. Core Features

> **Phase 1 functional requirements (FR IDs).** The IDs below cover **Phase 1 scope only**. Phase 2+ features (§3.3–3.7) receive FR IDs when those phases are specced for epics.

### FR-1 — Resort Discovery (Home) · Phase 1
**Maps to:** §3.1, Screen 1 (§12.5)

**Acceptance criteria:**
- [ ] All **20** seed resorts render as browsable cards with hero image, name, region, tagline, and badge summary
- [ ] Filter chips (region, skill level, snow quality, distance) and search by name update the list without full page reload
- [ ] Tapping a card navigates to the matching resort detail page (`resort_slug` in analytics)

### FR-2 — Resort Detail Page · Phase 1
**Maps to:** §3.2, Screen 2 (§12.5)

**Acceptance criteria:**
- [ ] Page shows hero image (static), quick verdict, highlights/lowlights, terrain data card, season-labelled T2 fields, and **link-out** to official `trail_map_url`
- [ ] Getting There content is reachable from the detail scroll or dedicated section (FR-4)
- [ ] Unsplash attribution renders on every hero image per §17.2

### FR-3 — Getting There · Phase 1
**Maps to:** §3.6, Screen 4 (§12.5)

**Acceptance criteria:**
- [ ] Per resort: ≥2 transport options with duration/cost estimate and external booking links (clean outbound, §11.1)
- [ ] Gateway airport city shows ≥3 experience cards where applicable
- [ ] Practical tips section (cash/card, onsen etiquette, JR Pass) is collapsible and readable on mobile

### FR-4 — Find My Resort Quiz · Phase 1
**Maps to:** Screen 0b (§12.5)

**Acceptance criteria:**
- [ ] 4-question sliding-card quiz accessible from onboarding and Home/Explore
- [ ] User can switch **Serious** (direct) and **Cosmic** (playful) question copy before Q1; default Serious
- [ ] Both modes map to the same four scoring enums — single `score.ts` algorithm (see [`features/quiz/brief.md`](../../features/quiz/brief.md))
- [ ] Completing the quiz shows a **podium of top 3 resorts** with mode-specific **results reveal** (Serious: header; Cosmic: interstitial + header + subline — `features/quiz/brief.md` § Results reveal + § Results podium)
- [ ] `quiz_started` and `quiz_completed` events fire per `docs/analytics/tracking-plan.md` (include `quiz_mode`, **`result_count: 3`**; `quiz_completed` after reveal completes)

### FR-5 — Onboarding · Phase 1
**Maps to:** Screen 0 (§12.5)

**Acceptance criteria:**
- [ ] First-time visitors see ≤4 swipeable cards; skip and "Get started" both work
- [ ] Onboarding is not shown again after completion (local flag or equivalent)
- [ ] No hero photography on onboarding cards (fast load)

### FR-6 — AI Chat (Resort Q&A) · Phase 1
**Maps to:** §4.2a, Screen 5 (§12.5)

**Acceptance criteria:**
- [ ] Floating "Plan with AI →" on resort detail opens full-screen chat
- [ ] Agent answers evergreen resort questions from seed content first (§4.1); live web search is **not** required in Phase 1
- [ ] §4.4 cost guardrails enforced (turn/token caps, per-IP limit, kill-switch)
- [ ] `ai_chat_opened` and `ai_chat_message_sent` events fire per tracking plan

### FR-7 — Info · Phase 1
**Maps to:** Info tab (§12.5)

**Acceptance criteria:**
- [ ] Info tab accessible from bottom nav with links to Privacy Policy and Terms of Service (§17.1)
- [ ] About copy explains product purpose; app version visible
- [ ] Contact / feedback method for data-rights requests (email sufficient for launch)

---

### 3.1 Feature 1 — Resort Discovery (Home) · Phase 1 · FR-1

**Purpose:** Present all **20** Japan ski destinations in a beautiful, browsable format. First impression must be visually stunning.

**Requirements:**
- Display resort cards with a hero photograph, resort name, region, and a 1-line tagline
- Filter/sort by: Region (Hokkaido, Tohoku, Nagano, Niigata, etc.), Skill Level (Beginner / Intermediate / Advanced), Snow Quality, Distance from Airport
- Search bar for resort name
- Each card shows a quick "badge" summary: e.g. 🏔 Varied terrain · ⭐ Powder heaven · 🚡 Ski-in ski-out
- Tapping a card opens the Resort Detail page
- Lazy-load images for performance; use WebP format

**The 20 Destinations (Phase 1 — all ship at launch):**

| # | Resort | Slug | Prefecture | Gateway Airport | Approx. Airport Distance |
|---|--------|------|------------|-----------------|--------------------------|
| 1 | Niseko United (Niseko) | `niseko-united` | Hokkaido | New Chitose (CTS) | ~2.5 hrs |
| 2 | Furano | `furano` | Hokkaido | New Chitose (CTS) | ~2 hrs |
| 3 | Rusutsu | `rusutsu` | Hokkaido | New Chitose (CTS) | ~1.5 hrs |
| 4 | Kiroro Snow World | `kiroro` | Hokkaido | New Chitose (CTS) | ~1.5 hrs |
| 5 | Sahoro (Club Med) | `sahoro` | Hokkaido | New Chitose (CTS) | ~2 hrs |
| 6 | Zao Onsen | `zao-onsen` | Yamagata | Sendai (SDJ) | ~1 hr |
| 7 | Appi Kogen | `appi-kogen` | Iwate | Hanamaki (HNA) | ~45 min |
| 8 | Hoshino Resorts Nekoma Mountain (formerly Alts Bandai) | `nekoma-alts-bandai` | Fukushima | Sendai (SDJ) | ~1.5 hrs |
| 9 | Hakuba Valley (×10 resorts) | `hakuba-valley` | Nagano | Matsumoto (MMJ) / Narita (NRT) | ~1–3 hrs |
| 10 | Shiga Kogen | `shiga-kogen` | Nagano | Narita (NRT) | ~3 hrs |
| 11 | Nozawa Onsen | `nozawa-onsen` | Nagano | Narita (NRT) | ~3 hrs |
| 12 | Myoko Kogen | `myoko-kogen` | Niigata | Narita (NRT) | ~3 hrs |
| 13 | Naeba | `naeba` | Niigata | Narita (NRT) | ~2 hrs |
| 14 | GALA Yuzawa | `gala-yuzawa` | Niigata | Tokyo Shinkansen direct | ~1.5 hrs from Tokyo |
| 15 | Madarao Kogen | `madarao-kogen` | Nagano | Narita (NRT) | ~3 hrs |
| 16 | Joetsu Kokusai | `joetsu-kokusai` | Niigata | Narita (NRT) | ~2.5 hrs |
| 17 | Tomamu (Hoshino) | `tomamu` | Hokkaido | New Chitose (CTS) | ~2 hrs |
| 18 | Lotte Arai Resort | `lotte-arai` | Niigata | Tokyo Shinkansen (Joetsumyoko) | ~2.5 hrs from Tokyo |
| 19 | Tsugaike Kogen (Hakuba) | `tsugaike-kogen` | Nagano | Matsumoto (MMJ) | ~1.5 hrs |
| 20 | Kandatsu Kogen | `kandatsu-kogen` | Niigata | Tokyo Shinkansen (Echigo-Yuzawa) | ~1.5 hrs from Tokyo |

---

### 3.2 Feature 2 — Resort Detail Page · Phase 1 · FR-2

**Purpose:** Give users all the information they need to evaluate and choose a resort.

**Sections:**

#### 2a. Hero & Overview
- Full-bleed **static hero image** (carousel optional; **no video loop in V1** — out of scope, §9)
- Resort name, prefecture, mountain stats (peak elevation, vertical drop, # of runs, # of lifts)
- Season dates (typical open / close) — **season-labelled** per §4.3
- Snow quality badge (e.g. "Champagne Powder", "Heavy Wet Snow")

#### 2b. Highlights & Lowlights
Present as two clearly styled sections. Use iconography for scannability.

**Highlight examples (per resort, curated):**
- Varied terrain (moguls, glades, groomers, park)
- Beginner-friendly (gentle slopes, wide runs)
- Ski-in ski-out accommodation available
- On-mountain restaurant(s)
- Top-tier gear rental on site
- Night skiing available
- World-class powder (especially Hokkaido)
- Proximity to hot spring / onsen town
- International signage / English-speaking staff
- Family-friendly facilities

**Lowlight examples (per resort, curated):**
- Remote location (long transfer required)
- Limited terrain variety (few runs or short vertical)
- Limited public transportation; car rental recommended
- Crowded during peak Japanese holidays
- No on-site accommodation
- Limited English support
- Expensive lift pass pricing
- Heavy snowfall can cause lift closures
- Limited après-ski options

#### 2c. Terrain Map · Phase 1
- **V1:** prominent link/button to official `trail_map_url` (opens resort's colour-coded piste map in new tab)
- **Future (Phase 4+):** embeddable interactive map (Fatmap/Slopes) if partnerships allow; not required for launch

#### 2d. Resort Data Card
| Field | Detail |
|-------|--------|
| Skiable area | e.g. 890 ha |
| Number of runs | e.g. 43 |
| Longest run | e.g. 5.6 km |
| Lifts | e.g. 22 (including gondolas) |
| Lift pass (adult/day) | e.g. ¥6,500 |
| Beginner % / Intermediate % / Advanced % | e.g. 30% / 40% / 30% |
| Average annual snowfall | e.g. 15m |
| Best months | e.g. January–February |
| Nearest town | e.g. Kutchan (Niseko) |
| Nearest village | e.g. Hirafu village |
| Nearest airport | e.g. New Chitose (CTS) |

---

### 3.3 Feature 3 — Accommodation Recommendations · Phase 2

**Trigger:** User selects one or more resorts (multi-select supported). Accommodation page generates per selection.

**Accommodation Types:**
- Ski-in / ski-out hotels and chalets (highlighted for beginners)
- Ryokan (traditional Japanese inn) — especially near onsen towns
- Budget options (guesthouses, hostels)
- Luxury / resort hotels

**Per Accommodation Card:**
- Name, type, price range (¥ / ¥¥ / ¥¥¥)
- Photo(s)
- Distance to ski lifts
- Key amenities (onsen, breakfast included, shuttle, gear storage)
- Direct booking link (external)
- Ski-in ski-out badge (prominent, especially for beginner users)

**Map Integration (Phase 2):**
- Show all recommended accommodations pinned on an interactive map — **Leaflet.js + OpenStreetMap default** (§7.1); Google Maps optional later
- Map also shows: resort lift base, nearby restaurants, rental shops
- Filter pins by type (accommodation / food / rental)

---

### 3.4 Feature 4 — Restaurant Recommendations · Phase 2

**Scope:** Near resort + in nearby village/town.

**Per Restaurant Card:**
- Name, cuisine type, price range
- Photo (sourced via Google Places API or curated)
- Star rating
- Reservation required? (Yes / No / Recommended)
- Link to reservation platform (Tabelog, OpenTable Japan, official site)
- Note if restaurant is on-mountain (visible from resort detail)

**Highlight local food context:**
- Hokkaido: fresh seafood, soup curry, dairy-rich dishes, ramen
- Nagano / Niigata: soba, sake, oyaki, hearty mountain cuisine
- Yamagata (Zao): imoni (taro stew), yamagata wagyu

---

### 3.5 Feature 5 — Gear Rental Recommendations · Phase 2

- Recommended rental shops near each resort (on-site and nearby)
- Name, distance, price range, equipment quality badge (Budget / Standard / Premium)
- Services offered: skis, snowboards, boots, helmets, kids gear, demo fleet
- Pre-booking link if available
- Tip: note if resort has on-site rental in Resort Detail

---

### 3.6 Feature 6 — Getting There (Trip Planning) · Phase 1 · FR-3

**Sections:**

#### 6a. Gateway Airport Guide
For each resort's gateway airport:
- Airport overview (size, terminals, international vs domestic connections)
- What to do in the airport city (e.g. Sapporo for CTS, Tokyo for NRT/HND, Sendai for SDJ)
  - Top 3–5 highlights in the city (sightseeing, food, culture)
  - Recommended 1-night pre/post trip stay options

#### 6b. Transportation to Resort
- Options: rental car, shuttle bus, train (JR), Shinkansen + local bus
- Estimated cost and duration for each
- Recommended option per resort (e.g. "Rental car recommended for Niseko from CTS")
- Link to booking platforms (Highway bus, JR Pass info, Nippon Car Rental etc.)

#### 6c. Check-in / Season Planning
- Typical resort open season (earliest / latest dates)
- Peak vs off-peak timing (avoid Golden Week, Japanese school holidays)
- Recommended trip length
- Typical daily schedule suggestion (lifts open/close times)

#### 6d. Practical Tips
- Japan Ski Pass (if resort is part of a multi-resort pass)
- IC Card / Suica for transit
- Cash vs credit card acceptance (many mountain restaurants are cash only)
- Onsen etiquette (tattoo policies, required towels)
- Language tips (basic Japanese ski vocabulary)
- Japan Weather App recommendations
- Travel insurance note

---

### 3.7 Feature 7 — Nearby Village & Town Explorer · Phase 2

**Purpose:** Extend the trip beyond the mountain. Users passionate about cultural exploration can discover:

- Village/town name, distance from resort
- Key experiences: onsen bathing, sake brewery tours, local markets, temples
- Food specialties
- Photo gallery
- Best time to visit (some towns are day-trip only, others worth an overnight)

**Example pairings:**
| Resort | Nearby Village/Town |
|--------|---------------------|
| Niseko | Kutchan, Otaru, Sapporo |
| Furano | Biei, Asahikawa |
| Zao Onsen | Yamagata City, Tendo (shogi pieces) |
| Hakuba | Matsumoto Castle town, Omachi |
| Nozawa Onsen | Nozawa village (free public baths), Togari |
| Myoko Kogen | Joetsu, Itoigawa (seafood on Sea of Japan) |
| GALA Yuzawa | Yuzawa town (sake museum, outlet mall) |

---

### 3.8 Phase 1 Improvements (Epic 6)

> **Post-launch amendment** — layered on Phase 1 after initial launch (Epic 6–8). Sharded view: `shards/16-phase1-improvements.md` etc.

**Source:** Product owner feedback post–Epic 5 · **Epic shard:** [`../../epics/phase1/shards/epic-06-phase1-improvements.md`](../../epics/phase1/shards/epic-06-phase1-improvements.md)  
**Timing:** After Phase 1 build complete (Epics 1–3, 5); **before** manual launch QA gate.

---

### Purpose

Phase 1 shipped a complete editorial product. Epic 6 captures **incremental improvements** — UX refinements, content quality, and ops — without expanding scope into Phase 2 (maps, accounts) or Phase 3 (AI, monetisation).

---

### Functional requirements (improvements)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-1** | Home resort list sorted by **popularity rank** (not alphabetical); **5** cards default; **Show all → Explore** (no in-page expansion) | 6.1 |
| **IMP-2** | Home **quiz CTA** below hero: "Find your right ski resort" + button to `/quiz` | 6.2 |
| **IMP-3** | Quiz **#1 podium card**: rank **"1"** overlay on image; remove side "Your top match" column | 6.3 |
| **IMP-4** | Quiz **back arrow** → `/explore` on all quiz phases | 6.4 |
| **IMP-5** | Trail map section: **button-only** outbound link; note text has **no raw URLs** | 6.5 |

---

### Non-functional / quality (retro backlog)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-6** | Highlight/lowlight columns: **16px icons** on column headers only (DESIGN.md polish) | 6.6 |
| **IMP-7** | **Unsplash images** for flagship resorts (Niseko, Hakuba, Nozawa minimum) | 6.7 |
| **IMP-8** | **≥3 daytrips** for GALA Yuzawa + Joetsu Kokusai | 6.8 |
| **IMP-9** | Verify/fix **broken external URLs** from launch link probe | 6.9 |
| **IMP-10** | **PostHog baseline dashboards** per tracking-plan §5 | 6.10 |
| **IMP-11** | **Production env documentation** (contact email, analytics keys) | 6.11 |
| **IMP-12** | Detail hero **quick verdict** smaller type + line clamp; Joetsu Kokusai name cleanup | 6.12 |

---

### Analytics changes (Epic 6)

| Event | Trigger | Phase | Story |
|-------|---------|-------|-------|
| `home_explore_cta_clicked` | Home "Show all resorts" → Explore | 1.1 | 6.1 |
| `quiz_cta_tapped` | Home hero quiz CTA (optional) | 1.1 | 6.2 |

**Deprecated behaviour:** Home in-page `load_more_clicked` — remove when 6.1 ships; Explore may retain load-more.

Update [`docs/analytics/tracking-plan.md`](../../../../docs/analytics/tracking-plan.md) in the same PR as each story.

---

### Out of scope (Epic 6)

- User accounts, login, saved trips (Phase 2+)
- Resort **user reviews** section → Phase 3 [`phase-3-user-community.md`](../../epics/phase1/shards/phase-3-user-community.md)
- AI chat, monetisation (Phase 3)
- Full 20-resort photography (6.7 Phase B — optional follow-up)

---

### Release gate

Epic 6 is **recommended complete** before executing [`docs/qa/phase1/launch-checklist.md`](../../../../docs/qa/phase1/launch-checklist.md) §3 manual QA. Minimum bar: **6.1–6.6 + 6.11**.


### 3.9 Phase 1 Improvements Wave 2 (Epic 7)

> **Post-launch amendment** — layered on Phase 1 after initial launch (Epic 6–8). Sharded view: `shards/16-phase1-improvements.md` etc.

**Source:** Product owner feedback post-launch · **Epic shard:** [`../../epics/phase1/shards/epic-07-phase1-ux-wave2.md`](../../epics/phase1/shards/epic-07-phase1-ux-wave2.md)  
**Timing:** After Epic 6 shipped to production (`main`, 2026-06-17).

---

### Purpose

Epic 6 closed pre-launch polish. Epic 7 captures **post-launch UX feedback** from real device usage — quiz flow, global spacing, navigation chrome, terrain chip readability, and geographically correct “Around resort” copy.

---

### Functional requirements (improvements)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-13** | Quiz questions **slide in-page** (no full-card remount); results **#1 visible** on iPhone SE | 7.1 |
| **IMP-14** | **Reduced top spacing**; responsive **content max-width** and horizontal padding on tablet/desktop | 7.2 |
| **IMP-15** | **Fixed overlay back button** top-left on quiz, detail, legal | 7.3 |
| **IMP-16** | Top bar: **no app title** on Home; **no “Find your resort” in bar**; **hamburger** + **inline search** (“What are you looking for”); Explore **h1 in page** | 7.4 |
| **IMP-17** | Terrain/mountain **stat chips ≤28 chars**; no horizontal chip scroll at 375px | 7.5 |
| **IMP-18** | Daytrips heading uses **`around_area`** (not airport); editorial **resort-area** daytrips | 7.6 |

---

### Analytics changes (Epic 7)

| Event | Trigger | Phase | Story |
|-------|---------|-------|-------|
| `nav_menu_opened` | Hamburger menu opened | 1.2 | 7.4 (optional) |
| `search_focused` | Inline search field focused | 1.2 | 7.4 (optional) |

Update [`docs/analytics/tracking-plan.md`](../../../../docs/analytics/tracking-plan.md) only if optional events ship.

**No change** to quiz funnel events for 7.1.

---

### Out of scope (Epic 7)

- Bottom nav item changes (still Home · Explore · Info)
- Global search beyond resort name (Phase 2+)
- AI chat, accounts, maps
- Rebranding / new app name in metadata (browser tab title may stay “Japan Ski Trip Planner”)

---

### Release gate

Epic 7 is **recommended complete** before the next manual QA pass. Minimum bar for merge: **7.2 + 7.6** (spacing + around-area bug). **7.1 + 7.4** are highest user-visible UX wins.


### 3.10 Quiz Cosmic Mode UX v2 (Epic 8)

> **Post-launch amendment** — layered on Phase 1 after initial launch (Epic 6–8). Sharded view: `shards/16-phase1-improvements.md` etc.

**Created:** 2026-06-17 · **Updated:** 2026-06-17  
**Status:** PO decisions locked — ready for implementation  
**Source:** Product owner feedback post–Epic 7 launch  
**Epic shard:** [`../../epics/phase1/shards/epic-08-quiz-cosmic-story.md`](../../epics/phase1/shards/epic-08-quiz-cosmic-story.md)  
**Feature brief (baseline):** [`features/quiz/brief.md`](../../features/quiz/brief.md)  
**Asset guide:** [`content/quiz-illustrations/README.md`](../../../../content/quiz-illustrations/README.md)  
**Timing:** After Epic 7 on `main`. Serious mode unchanged unless noted.

---

### 0. Document purpose

This PRD defines the **Cosmic mode refresh** for the Find My Resort Quiz: the **Snow Forest Path** story arc, per-question illustrations, mystical answer buttons with sparkle motion, and a **floating candle ambience** on the existing light app background. Scoring enums and `score.ts` stay **unchanged** — only display copy, visuals, and Cosmic-only chrome ship here.

---

### 0.1 Product decisions (locked)

| Topic | Decision | Notes |
|-------|----------|-------|
| **Mode label** | Keep **Cosmic** | Subtitle under toggle when Cosmic selected: *“Let the universe choose for me”* |
| **Story arc** | **Option A — Snow Forest Path** | Four linked scenes; copy in §6 |
| **Page background** | **Light** (`--color-bg` / `#f7f5f2`) | No full dark cosmic stage in v1; keeps app clean and consistent |
| **Ambient motion** | **Floating lighted candle** near question block | SVG/CSS; gentle float + flicker; forest-lantern motif |
| **Scene illustrations** | **Static WebP**, one per question | AI-generated with locked style prompt (primary); free licensed set as fallback |
| **Attribution** | **Required** where license demands | Credits in Info tab + `content/quiz-illustrations/README.md` |
| **Analytics enum** | **`quiz_mode: cosmic`** unchanged | `story_arc_version: "forest-v1c"` on quiz events |
| **Deferred** | Full dark cosmic stage | PO may revisit; do not implement in Epic 8 |

---

### 1. Problem statement

| Issue | Evidence | Impact |
|-------|----------|--------|
| **“Cosmic” is opaque** | Toggle label alone doesn’t explain the mode | Users skip Cosmic or pick it without knowing what to expect |
| **Questions feel disconnected** | Q1 = phone call; Q2 = evening; Q3 = postcard; Q4 = bookstore | No progression; hard to stay engaged |
| **Visually flat** | Same white buttons as Serious; no imagery ([`features/quiz/brief.md`](../../features/quiz/brief.md) § UX notes) | Cosmic feels like a text swap |
| **Weak ski-trip link** | Q1 park/phone metaphor doesn’t evoke Japan winter travel | Playful mode under-delivers on trip planning |

**Serious mode** remains the default, direct path. This epic only elevates **Cosmic**.

---

### 2. Vision

Cosmic mode becomes a **short illustrated winter fable** — *The Snow Forest Path* — four scenes, one decision each, same scoring underneath. The page stays **clean and light** like the rest of the app; magic comes from **scene art**, **twilight-colored answer buttons with sparkles**, and a **soft floating candle** that drifts beside the question like a forest lantern guiding the way.

Users who want the universe to choose get a memorable, slightly mystical journey; users who want speed keep Serious.

---

### 3. Target user

#### Jobs to be done

- **Undecided planner (Alex):** “Make the quiz fun enough that I finish it instead of bouncing to filters.”
- **Repeat visitor (Mind):** “Give me a reason to try Cosmic after Serious.”
- **Cosmic-curious user:** “I don’t know what Cosmic means — the subtitle should tell me it’s hands-off / fun.”

#### User journey — UJ-8. The forest path

- **Persona + context:** First-time visitor, tapped “Find my resort” from Home.
- **Entry state:** `/quiz` intro; **Serious** (default) and **Cosmic** with subtitle *Let the universe choose for me*.
- **Path:** Chooses Cosmic → Scene 1 illustration + fork question + candle ambience → colored sparkle buttons → slide to scenes 2–4 → path-reading interstitial → podium top 3.
- **Climax:** Results header ties back to the forest (“The path led you here…”).
- **Resolution:** Taps #1 resort or Browse all → Explore.
- **Edge case:** `prefers-reduced-motion: reduce` → no sparkles, no candle motion; static candle icon optional.

---

### 4. Glossary

- **Cosmic mode** — User-facing label and internal enum `quiz_mode: cosmic` (unchanged in PostHog).
- **Snow Forest Path** — Approved story arc (`story_arc_version: forest-v1c`); four scenes in §6.
- **Scene illustration** — One static image per question; same visual style across all four.
- **Candle ambience** — Decorative floating lantern/candle SVG near the question; CSS animation only.
- **Sparkle layer** — Particle shimmer on Cosmic answer button tap; decorative only.
- **Scoring enum** — `level`, `priority`, `region`, `timing` — unchanged; see [`features/quiz/scoring.md`](../../features/quiz/scoring.md).

---

### 5. Narrative — Snow Forest Path (approved)

**Premise:** You wander into a quiet snow forest near the Japanese mountains. Each question is the next step on the path.

| Q | Axis | Scene | Narrative beat |
|---|------|-------|----------------|
| 1 | `level` | Fork in the trail | Which slope/path matches your comfort |
| 2 | `priority` | Valley clearing at dusk | What makes the perfect evening |
| 3 | `region` | Three spirit gates in the snow | Which regional gate calls to you |
| 4 | `timing` | Stone winter clock in a clearing | When your journey begins |

#### Alternatives considered (not shipping)

| Option | Why not chosen |
|--------|----------------|
| B — Powder Oracle’s Cabin | Indoor scenes risk visual sameness |
| C — Night Train North | Weaker Q1 skill metaphor |
| Rename Cosmic → Story | PO prefers **Cosmic** + subtitle |
| Full dark cosmic stage | PO prefers light, clean page; deferred |

---

### 6. Copy — Snow Forest Path

Implement in `web/messages/en.json` under `quiz.cosmic.*`. Option `id` keys **unchanged** (scoring stable).

**Copy principle:** Answer choices stay inside the forest fable — not ski jargon. Each option **implies** the scoring enum without naming runs or slopes.

**Status:** **Locked** — PO mixed A/B selections (2026-06-17). Ship **`forest-v1c`** below. A/B drafts kept in §6.1 for reference only.

#### Mode toggle & intro

| Element | Copy | i18n key |
|---------|------|----------|
| Mode label | Cosmic | `quiz.modeCosmic` (existing) |
| Mode subtitle (Cosmic active) | Let the universe choose for me | `quiz.modeCosmicSubtitle` |
| Cosmic intro line (optional, below toggle) | Follow the snow forest path — four scenes, one resort match. | `quiz.introCosmic` |
| Scene progress | Scene {current} of {total} | `quiz.sceneProgress` (Cosmic only; Serious keeps `quiz.progress`) |

---

#### Final copy — `forest-v1c` *(PO approved · ship in 8.1)*

##### Scene 1 — `level` (fork in the trail)

**Question:** *You reach a fork in the snow-laced trail. A wooden sign reads: “Choose your path.”*

| Option id | Copy |
|-----------|------|
| `beginner` | You take the lantern-lit path — slow and easy. |
| `confident` | The worn path looks safe; you'll try it and see how it goes. |
| `advanced` | A narrow opening in the pines — only you noticed it — and you slip through. |
| `group` | You wait at the sign for the others — you'll follow their lead. |

##### Scene 2 — `priority` (valley at dusk)

**Question:** *The forest opens to a valley. What would make tonight perfect?*

| Option id | Copy |
|-----------|------|
| `powder_terrain` | You stay out in the quiet snow — just you and the mountains. |
| `village_culture` | You walk toward the village lights, food, and narrow streets. |
| `family` | Smoke rises from a hut where voices sound safe — you head toward the fire. |
| `all` | Why choose one ending? You'll take the long way and have a little of everything. |

##### Scene 3 — `region` (spirit gates)

**Question:** *Three gates stand in the snow. Which one pulls you through?*

| Option id | Copy |
|-----------|------|
| `hokkaido` | The far gate — wide open snow and cold northern wind. |
| `honshu` | The middle gate hums like distant rails; steam curls up from a bowl you can't see yet. |
| `tohoku` | The northern gate breathes hot spring mist against the cold air. |
| `unsure` | You don't walk through any gate — you're not ready to choose. |

##### Scene 4 — `timing` (stone winter clock)

**Question:** *You find a stone clock in the snow. Four faces mark different depths of winter. When does your journey begin?*

> Replaces “frozen sundial” — easier to picture; choice copy unchanged (still uses *face* / *hand*).

| Option id | Copy |
|-----------|------|
| `early_season` | The face of thin light — early footsteps, an empty world still waking. |
| `peak_february` | The face locked in deepest winter — the hour when everything holds still. |
| `late_season` | The face where snow softens and the shadows grow long. |
| `still_planning` | The hand keeps turning — you'll know when it finally stops. |

**Analytics:** `story_arc_version: "forest-v1c"` on `quiz_started` / `quiz_completed` when `quiz_mode: cosmic`.

---

#### §6.1 Draft variants (reference only — do not ship)

<details>
<summary>Variant A / B drafts used during PO review</summary>

Questions identical except Scene 4 (v1c question above).

##### Scene 1 — `level` (fork in the trail)

**Question:** *You reach a fork in the snow-laced trail. A wooden sign reads: “Choose your path.”*

| Option id | Copy (A) |
|-----------|----------|
| `beginner` | The lantern path dims and brightens slowly — you trust its unhurried glow. |
| `confident` | One trail looks well-walked; you'll follow it once before going further. |
| `advanced` | A narrow opening in the pines — only you noticed it — and you slip through. |
| `group` | You linger at the sign until footsteps catch up behind you. |

##### Scene 2 — `priority` (valley at dusk)

**Question:** *The forest opens to a valley. What would make tonight perfect?*

| Option id | Copy (A) |
|-----------|----------|
| `powder_terrain` | You keep walking until the forest goes quiet and the day settles in your bones. |
| `village_culture` | Lantern-light flickers between distant roofs — you drift toward the warmth and broth-steam. |
| `family` | Smoke rises from a hut where voices sound safe — you head toward the fire. |
| `all` | Why choose one ending? You'll take the long way and have a little of everything. |

##### Scene 3 — `region` (spirit gates)

**Question:** *Three gates stand in the snow. Which one pulls you through?*

| Option id | Copy (A) |
|-----------|----------|
| `hokkaido` | The farthest gate — open white silence and a distant cowbell on the wind. |
| `honshu` | The middle gate hums like distant rails; steam curls up from a bowl you can't see yet. |
| `tohoku` | The northern gate breathes hot spring mist against the cold air. |
| `unsure` | None of the gates feel ready — you wait in the grey between them. |

##### Scene 4 — `timing` (frozen sundial)

**Question:** *A frozen sundial shows four faces. When does your journey begin?*

| Option id | Copy (A) |
|-----------|----------|
| `early_season` | The face of thin light — early footsteps, an empty world still waking. |
| `peak_february` | The face locked in deepest winter — the hour when everything holds still. |
| `late_season` | The face where snow softens and the shadows grow long. |
| `still_planning` | The hand keeps turning — you'll know when it finally stops. |

---

#### Variant B — Clear story *(recommended · easier to read · still Cosmic)*

Same four questions. Shorter words; each choice reads as a plain story action.

##### Scene 1 — `level`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `beginner` | You take the lantern-lit path — slow and easy. | Just starting / gentle |
| `confident` | The worn path looks safe; you'll try it and see how it goes. | Getting comfortable |
| `advanced` | You spot a hidden gap in the trees and go through on your own. | Bold / experienced |
| `group` | You wait at the sign for the others — you'll follow their lead. | Mixed group |

##### Scene 2 — `priority`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `powder_terrain` | You stay out in the quiet snow — just you and the mountains. | Powder & terrain |
| `village_culture` | You walk toward the village lights, food, and narrow streets. | Village & culture |
| `family` | You join the group by a warm fire — everyone together tonight. | Family-friendly |
| `all` | You want a bit of everything — snow, town, and good company. | All of the above |

##### Scene 3 — `region`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `hokkaido` | The far gate — wide open snow and cold northern wind. | Hokkaido |
| `honshu` | The middle gate — big mountains and a train far in the distance. | Honshu |
| `tohoku` | The gate in rising steam, like a hot spring town in winter. | Tohoku |
| `unsure` | You don't walk through any gate — you're not ready to choose. | Not sure yet |

##### Scene 4 — `timing`

| Option id | Copy (B) | What it means |
|-----------|----------|---------------|
| `early_season` | Early winter — the season is just starting, not many people around. | Dec–Jan |
| `peak_february` | Mid-winter — deep snow and the busiest time on the mountain. | Peak Feb |
| `late_season` | Late winter — softer snow and warmer, longer days. | Mar–Apr |
| `still_planning` | The dial still spins — you haven't picked your dates yet. | Still planning |

</details>

---

### 7. Visual design — Cosmic chrome (light page)

Applies **only** when `quiz_mode === cosmic`. **Page background stays `--color-bg`** (#f7f5f2). Serious mode unchanged.

#### 7.1 Layout — `ResortQuizCard` (Cosmic)

```
┌─────────────────────────────────────┐  ← --color-bg (light, unchanged)
│  [Scene illustration 16:10]         │
│  Scene 2 of 4                       │
├─────────────────────────────────────┤
│      🕯️ ← floating candle (ambient) │
│  Question (serif heading)           │
│  ┌───────────────────────────────┐  │
│  │ Answer A (twilight violet)  ✨│  │
│  ├───────────────────────────────┤  │
│  │ Answer B (aurora teal)        │  │
│  …                                  │
└─────────────────────────────────────┘
```

- Illustration band: full content width, rounded `--radius-card`, subtle shadow optional
- Candle: positioned **upper-right or left of question** (implementation picks one; avoid overlapping question text on 375px)
- Serious: no illustration, no candle, standard progress copy

#### 7.2 Cosmic palette (CSS tokens)

Add to `tokens.css` under `[data-theme='default']`:

| Token | Hex | Use |
|-------|-----|-----|
| `--color-cosmic-opt-1` | `#4a3f8c` | Answer 1 — twilight violet |
| `--color-cosmic-opt-2` | `#2d6b8a` | Answer 2 — aurora teal |
| `--color-cosmic-opt-3` | `#6b4a7a` | Answer 3 — dusk plum |
| `--color-cosmic-opt-4` | `#8a6b3a` | Answer 4 — star gold |
| `--color-cosmic-sparkle` | `#e8e0ff` | Sparkle particle highlight |
| `--color-cosmic-candle-glow` | `#f5d78e` | Candle flame / glow |
| `--color-cosmic-on-opt` | `#ffffff` | Button label on colored buttons |

**Not used in v1:** `--color-cosmic-bg` dark stage tokens (reserved for future experiment).

#### 7.3 Answer buttons

| Aspect | Serious (unchanged) | Cosmic |
|--------|---------------------|--------|
| Layout | Full-width, min 52px | Same |
| Background | `surface` + divider border | `--color-cosmic-opt-N` by option index (1–4) |
| Text | `text-primary` | `--color-cosmic-on-opt` |
| Border | 1px divider | None or 1px rgba white 20% |
| Press | `scale(0.99)` | Same + sparkle burst |

Map colors by **option order** (not enum value) for visual balance every question.

#### 7.4 Sparkle animation

| Rule | Spec |
|------|------|
| Trigger | On `:active` / tap |
| Technique | CSS pseudo-elements or inline SVG — **no Lottie in v1** |
| Duration | 400–600ms ease-out |
| Particles | 6–10 points along button top edge; fade + translate |
| Reduced motion | Disabled entirely; solid color press state only |
| Performance | `transform` + `opacity` only |

#### 7.5 Floating candle ambience

| Rule | Spec |
|------|------|
| Asset | Inline **SVG** lantern or candle (single component `CosmicCandleAmbience`) |
| Placement | Near question heading; `position: absolute` within question block wrapper |
| Float | Slow vertical drift ±6px, ~4s loop, ease-in-out |
| Flicker | Flame opacity 85–100%, ~1.2s irregular loop (two keyframes enough) |
| Size | ~32–40px wide; must not reduce question tap area |
| Reduced motion | Static SVG at rest position; no animation |
| z-index | Below back button; above background only |

Candle reinforces forest-lantern motif from Scene 1 without darkening the page.

#### 7.6 Scene illustrations

| # | Scene | Visual brief |
|---|-------|--------------|
| 1 | **The fork** | Snow trail splits; lantern on left path; darker trees on right; no people |
| 2 | **The clearing** | Wide valley, dusk sky, distant village lights vs open snow |
| 3 | **The gates** | Three Nordic stone arch gates (carved rock, Frozen 2–inspired); subtle regional hints (fields / train / steam) |
| 4 | **The stone clock** | Old stone clock in snow; four faces suggesting early / deep / late winter + turning hand |

**Style direction (locked across all four):**

- Medium: soft gouache / editorial watercolor — **not** anime, **not** 3D, **not** photoreal
- Palette: muted winter — cream `#f7f5f2`, slate blue, pine green, touch of `#c4873a` (accent-warm)
- Line: consistent ink weight; no text in image
- Format: **WebP**, 800×500 logical (~1600×1000 export), **≤120KB** each
- Path: `web/public/quiz/cosmic/q1.webp` … `q4.webp`
- Delivery: `next/image` with `priority` on current scene only

#### 7.7 Illustration sourcing (locked)

**Primary — AI with locked style**

1. Write one **style anchor prompt** (§15) and reuse for all four scenes; vary **scene prompt** only.
2. Generate 2–3 candidates per scene; PO picks one set where all four feel same “artist.”
3. Post-process: crop, compress WebP, verify file size budget.
4. Document model + prompts in [`content/quiz-illustrations/README.md`](../../../../content/quiz-illustrations/README.md).

**Fallback — free licensed illustration set**

Use only if AI batch fails consistency QC after 2 rounds:

| Source | License | Cost | Notes |
|--------|---------|------|-------|
| [Storyset](https://storyset.com) | Free with attribution | $0 | Recolor to brand palette |
| [unDraw](https://undraw.co) | Free, no attribution required | $0 | May feel generic; customize colors |
| [Open Doodles](https://opendoodles.com) | CC0 | $0 | Looser style; harder to match Kinfolk tone |

**Attribution:** If required, add **Illustrations** line on Info tab (`info.illustrationsCredit`) linking to artist/source. Mirror full credits in `content/quiz-illustrations/README.md`.

---

### 8. Results reveal (Cosmic)

Enum/scoring unchanged. Update strings only:

| Step | Was | Now |
|------|-----|-----|
| Interstitial | Consulting the powder oracle… | *Reading the signs on the path…* |
| Header | The stars have spoken. Here are your resorts: | *The path led you here — your top resorts:* |
| Subline | Ranked by cosmic vibes and extremely real resort data. | *Chosen by the universe — backed by real resort data.* |
| #1 label | Your cosmic #1 | *Your cosmic #1* (keep — label still fits PO tone) |

i18n keys: `quiz.reveal.cosmic.loading`, `quiz.results.cosmic`, `quiz.results.cosmicSubline`, `quiz.results.topMatchLabelCosmic`.

Interstitial timing: **~800ms** crossfade (unchanged).

---

### 9. Functional requirements

| ID | Requirement |
|----|-------------|
| **FR-4a** | Cosmic shows **one scene illustration per question** (4 assets), consistent style |
| **FR-4b** | Cosmic answer buttons use **four distinct palette colors** |
| **FR-4c** | Cosmic buttons show **sparkle on tap**; off when reduced motion |
| **FR-4d** | Cosmic copy is **Snow Forest Path** (§6); enum mapping unchanged |
| **FR-4e** | Toggle shows **Cosmic** + subtitle *Let the universe choose for me*; `quiz_mode: cosmic` in events |
| **FR-4f** | Serious mode **unchanged** |
| **FR-4g** | Results reveal copy per §8 |
| **FR-4h** | **Floating candle ambience** on Cosmic question screens; static when reduced motion |
| **FR-4i** | Page background **remains light** (`--color-bg`); no full dark stage |
| **FR-4j** | `story_arc_version: "forest-v1c"` on `quiz_started` and `quiz_completed` |

#### Acceptance criteria (testable)

- [ ] Cosmic toggle shows subtitle when Cosmic selected (or always visible under Cosmic pill — implementer choice; must be readable)
- [ ] Scenes 1–4: illustration + scene progress + candle + 4 colored buttons on 375px
- [ ] Ninja **Serious** path → Rusutsu/Kiroro/Niseko top 3 (`features/quiz/scoring.md` §6)
- [ ] Cosmic enum answers → **identical podium** to pre–Epic 8 cosmic mapping
- [ ] `quiz_started` / `quiz_completed` include `quiz_mode: cosmic` + `story_arc_version: forest-v1c`
- [ ] Reduced motion: no sparkle, no candle float/flicker
- [ ] Illustration total ≤500KB; LCP regression ≤200ms vs pre–Epic 8 baseline
- [ ] Illustration attribution documented; Info credit if license requires

---

### 10. Non-goals (Epic 8)

- Scoring weights or 5th question
- Illustrations or candle on **Serious** mode
- **Full dark cosmic stage** (deferred — PO may revisit)
- Sound / haptics
- **Animated scene illustrations** (GIF/video/Lottie for scenes)
- 16 per-answer illustrations
- Renaming `cosmic` in analytics schema
- Thai Cosmic copy in v1 (English only; Serious Thai unchanged)

---

### 11. Success metrics

| ID | Metric | Target |
|----|--------|--------|
| **SM-8a** | Cosmic selection rate | ≥25% of quiz starts (baseline from PostHog after 2 weeks) |
| **SM-8b** | Cosmic completion rate | ≥ Serious completion within 4 weeks |
| **SM-8c** | Quiz → detail tap (Cosmic) | ≥1 podium tap per completed Cosmic quiz (median) |

**Counter-metric:** Do not optimize time-on-quiz if completion drops.

---

### 12. Open questions

1. **Thai locale:** Ship Cosmic copy English-only in Epic 8, or hide Cosmic until translated? *(Default: English Cosmic OK in v1.)*
2. **Subtitle placement:** Under toggle only when Cosmic active vs always visible under Cosmic segment? *(Design spike in 8.4.)*

---

### 13. Assumptions

- AI style lock achieves 4 cohesive scenes within 2 generation rounds.
- Floating candle reads as magical on light bg without cluttering SE viewport.
- Free licensed fallback acceptable if AI QC fails.

---

### 14. See also

| Need | Document |
|------|----------|
| Epic stories | [`../../epics/phase1/shards/epic-08-quiz-cosmic-story.md`](../../epics/phase1/shards/epic-08-quiz-cosmic-story.md) |
| Illustration pipeline | [`content/quiz-illustrations/README.md`](../../../../content/quiz-illustrations/README.md) |
| Baseline quiz spec | [`features/quiz/brief.md`](../../features/quiz/brief.md) |
| Scoring | [`features/quiz/scoring.md`](../../features/quiz/scoring.md) |
| Motion rules | [`14-ux-components-motion.md`](shards/14-ux-components-motion.md) |

---

### 15. Appendix — AI style lock prompt (v1)

Use the **same style block** for every scene; only change the scene description line.

**Style anchor (append to every generation):**

> Soft editorial watercolor illustration, muted Japanese winter palette, cream and slate blue and pine green, gentle gouache texture, thin consistent ink outlines, calm Kinfolk magazine aesthetic, no anime, no 3D, no photorealism, no people, no text, no logos, horizontal composition, plenty of negative space.

**Scene prompts:**

| File | Scene description |
|------|-------------------|
| `q1.webp` | Snow-covered forest trail splitting into two paths, wooden signpost, paper lantern glow on the gentler left path, dark pine trees, peaceful mood |
| `q2.webp` | Forest opening to wide snowy valley at dusk, soft purple sky, tiny warm village lights in far distance, open powder field foreground |
| `q3.webp` | Three snow-covered Nordic stone arch gates in a row, carved rock pillars, subtle hints: open white field, distant train silhouette, rising onsen steam — not torii |
| `q4.webp` | Old stone clock half-buried in snow, four clock faces or seasonal carvings suggesting winter stages, mystical but minimal, turning hand optional |

**QC checklist:** Same line weight, same sky treatment, same snow texture, no human figures, exports compress to ≤120KB WebP each.


---

## 4. AI-Powered Content — Search & Chat Agent

### 4.1 Overview
To keep information fresh and add intelligent planning assistance, integrate a Claude-powered AI agent directly in the app.

**Default behaviour — seed-content first, model second.** The agent's primary knowledge source is the structured resort content the rest of the app already uses (`content/resorts/<slug>.md`, §8.1). The order of operations for any query is:

1. **Answer from seed content** wherever possible — most questions ("is it good for beginners?", "best months?", "how do I get there?") are already evergreen facts in the resort files. Resolve these with retrieval (and, if needed, a cheap model), not a premium generation call.
2. **Call the model for synthesis** only when the task genuinely needs it — trip planning, comparisons, multi-step reasoning.
3. **Trigger a live web search only for live (T3) data** — current snow, lift status, today's prices — and serve it from the shared TTL cache, not a per-user search (§4.4.2).

This keeps answers fast, consistent with what's shown on the page, and cheap to run (see §4.4 and §14). The model should never be the first resort for information that already exists in seed content.

### 4.2 Use Cases

> **Phase mapping:** **4.2a → Phase 1** (FR-6). **4.2b → Phase 3** (full wizard, requires Plan tab + saved trips). **4.2c → Phase 3** (live T3 web search).

**4.2a "Ask About a Resort" Chat · Phase 1**
- Embedded conversational interface on each Resort Detail page
- Users can ask: "Is Niseko good for beginners?" / "What's the best time to visit Hakuba for powder?" / "How do I get from CTS to Niseko without a car?"
- Agent has access to resort seed data. **Live web search is Phase 3** — Phase 1 answers from seed content + model synthesis only.

**4.2b "Plan My Trip" Flow · Phase 3**
- Wizard-style chat that collects: skill level, dates, group composition, budget, interests (resort-focused vs culture-heavy)
- Agent outputs a curated trip itinerary: resort recommendation + accommodation + day-by-day plan + transport summary

**4.2c Live Snow & Conditions Search · Phase 3**
- "Current snow conditions" query triggers a web search tool call to pull latest snow report from resort's official site or snow-forecast.com
- Show: base depth, fresh snow (last 24h / 72h), open runs %, next forecast

### 4.3 Data Freshness Strategy
- Seed data (resort facts, highlights/lowlights, terrain stats): managed in a CMS (e.g. Contentful or Notion-backed), updated once per season by an admin or editorial workflow
- Dynamic data (snow reports, lift pass prices, events): fetched live via AI agent web search at query time
- Restaurant/accommodation data: sourced from Google Places API, refreshed periodically via scheduled job

**Season currency policy (product behaviour).** Content follows the volatility tiers in `content/content-model.md` (T1 evergreen / T2 seasonal / T3 live). The product **always shows the latest *available* season and never blocks on data that does not exist yet:**
- **Off-season (e.g. now):** show the most recent published season for seasonal (T2) fields — lift-pass prices, confirmed open/close dates — and **always label the season explicitly** (e.g. "¥X — 2025/26 season"). Never present a stale number as if it were current. *Current baseline: 2025/26.*
- **Pre-season refresh (~Sep–Nov):** update T2 fields to the upcoming season (e.g. 2026/27) as resorts publish them. Evergreen (T1) facts stay untouched unless something physically changed (e.g. a new lift).
- **Latest-available wins:** if a resort has already published next-season (2026/27) figures, use them immediately rather than waiting for the refresh window.
- **Live (T3) conditions** — snow depth, today's lift status, real-time pricing — are never stored; the AI agent / Places API fetch them at query time.
- The UI must surface the season label and a "last reviewed" signal so users can trust what they're seeing (ties to §11.3 Resort Data Accuracy).

### 4.4 AI Cost Management & Guardrails

The AI chat is **free to users but not free to run**. Claude API calls — especially with web search enabled — scale linearly with usage and can spike without warning if the product gets popular or is abused. Because there is **no monetisation in V1** (§11.1), the AI feature must be **bounded by design**, not by hope. These guardrails are requirements for Phase 3, not optional polish.

> All prices below are **planning estimates** and must be re-validated against current [Anthropic pricing](https://www.anthropic.com/pricing) before launch. Order of magnitude is what matters here.

#### 4.4.1 Hard limits (must implement)

| Control | Limit (starting point) | Behaviour at limit |
|---------|------------------------|--------------------|
| **Max turns per session** | 10 user messages | Show "Let's start a fresh chat to keep things snappy" + offer to summarise; reset context |
| **Max input per message** | ~1,000 tokens (~750 words) | Reject/trim with a friendly prompt to shorten |
| **Max output per response** | `max_tokens` = 600–800 | Hard cap on the API call; responses stay concise (also better UX) |
| **Conversation context cap** | Last ~6 turns or a rolling summary | Summarise older turns rather than resending the full transcript every call |
| **Web searches per turn** | 1 (2 max) | Only triggered for genuinely live intents (snow, conditions, current prices) |
| **Web searches per session** | 3 | Further "live" asks answered from the shared cached snow data (§7.3) |
| **Daily cap per anonymous ID / IP** | ~30 messages or 5 sessions/day | Rate-limit with a graceful message; protects against abuse & runaway cost |
| **Global daily spend kill-switch** | A budget threshold you set | Above it, degrade gracefully to **non-AI fallback** (canned answers + seed-content search), never unbounded spend |

#### 4.4.2 The biggest cost lever: don't call the model when you don't need to

- **Evergreen questions are free to answer from seed content.** "Is Niseko good for beginners?" is already in the resort `.md` files (§8.1). Answer from structured content / a cheap retrieval step — reserve a full Claude call for genuine synthesis or planning.
- **Live snow search must be a *shared, cached* fetch — not per-user.** Fetch each resort's conditions once and cache 6–24h (§7.3 already specifies TTL); every user in that window reads the cache. This converts "N users × 1 search each" into "1 search per resort per few hours" — often a 10–100× reduction.
- **Pre-generate answers** to the top handful of recurring questions per resort from seed content; serve them with zero live calls.

#### 4.4.3 Token & model discipline (per call)

- **Prompt caching** for the static system prompt + per-resort context block (reused across every turn on a resort page) — cache reads are ~10% the cost of fresh input. This is the single highest-leverage optimisation for chat.
- **Model routing:** use a small/cheap model (e.g. Claude Haiku) for intent classification ("does this even need web search?"), routing, and simple FAQ; reserve the larger model (Sonnet, §7.4) for trip-planning synthesis. Most turns don't need the expensive model.
- **Retrieve, don't dump:** send only the relevant resort fields to the model, not the whole content file.
- **Tight `max_tokens`** on every call; stream for UX but keep the cap.

#### 4.4.4 Estimated cost per active user (worked example — validate before launch)

Assumptions: Sonnet-class pricing (~$3 / 1M input, ~$15 / 1M output), web search ~$10 / 1k searches, prompt caching on.

| Item | Estimate |
|------|----------|
| Typical chat turn (cached context, no search) | ~$0.005–0.01 |
| Turn that triggers a (shared-cache) answer | ~same — no per-user search |
| Turn that triggers a live web search | ~$0.03–0.04 |
| **Engaged chat session** (~6 turns, ≤1 live search) | **~$0.05–0.10** |
| If ~30% of monthly users open chat (~1 session each) | **~$0.02–0.03 per active user** |

Illustrative monthly totals (with caps in place):

| MAU | Chat users (30%) | Est. AI cost / month |
|-----|------------------|----------------------|
| 10,000 | 3,000 | ~$150–300 |
| 50,000 | 15,000 | ~$750–1,500 |
| 100,000 | 30,000 | ~$1,500–3,000 |

#### 4.4.5 Phase 3 viability verdict

With the guardrails above — especially **shared-cached snow data, prompt caching, model routing, and answering evergreen questions from free seed content** — the AI feature is **financially viable without monetisation at small-to-moderate scale** (low hundreds of dollars/month at ~10k MAU). The daily spend kill-switch (§4.4.1) makes the **worst case bounded**: cost can never run away, it just degrades to the non-AI fallback. Revisit this once real usage data exists (tie to analytics §13) and before any large marketing push. See §14 for run-time and build-time optimisation guidelines, and §11.1 for the monetisation stance.

---

## 5. UI/UX Design Requirements

> **Superseded by §12 for implementation.** This section captures early intent. **§12 UI/UX Specification is canonical** for typography, colours, navigation, and screen specs. Where §5 and §12 conflict, follow §12.

### 5.1 Visual Style
- **Aesthetic:** Minimal white, editorial, premium travel magazine feel
- **Color palette:** Off-white background (#FAFAFA), deep charcoal text (#1A1A1A), single accent color (icy blue or mountain slate)
- **Typography:** See §12.3 (Default Theme: Cormorant Garamond + DM Sans). Theme B (Noto Serif JP + Noto Sans JP) activates with TH/JP i18n (Phase 2–3).
- **Photography:** High-quality hero images (Unsplash in V1, §11.2). Static images only — no hero video in V1.
- **Whitespace:** Generous padding; let images breathe
- **Icons:** Minimal line icons (Lucide or Phosphor icon set)

### 5.2 Mobile-First Layout
- Designed for 375–430px viewport width (iPhone SE → iPhone Pro Max)
- **Bottom navigation (Phase 1):** Home · Explore · Info — see §12.5 for phase-tagged nav evolution
- Sticky header with resort name on detail pages
- Swipeable card stacks for accommodation and restaurant browsing *(Phase 2)*
- Map view full-screen toggle *(Phase 2)*

### 5.3 Key Screens

| Screen | Phase | Description |
|--------|-------|-------------|
| Home / Discovery | 1 | Grid of all 20 resort cards with hero photos, filter bar |
| Onboarding | 1 | First-time swipeable intro cards |
| Find My Resort (quiz) | 1 | 4-question sliding quiz → filtered list |
| Resort Detail | 1 | Scrollable detail: highlights, data card, getting there, AI entry |
| Getting There | 1 | Transport options, gateway city guide *(may live inside detail scroll)* |
| AI Chat | 1 | Floating button → full-screen resort Q&A |
| Info | 1 | About, legal links, feedback contact |
| Accommodations | 2 | Filtered list + map of hotels/ryokan |
| Restaurants | 2 | Grid + map of nearby dining |
| Village Explorer | 2 | Cultural attractions near resort |
| Resort Comparison | 3 | Side-by-side comparison of up to 3 resorts |
| Plan / My Trip | 3 | Saved resorts, accommodation, itinerary workspace |

### 5.4 Performance Targets
- First Contentful Paint: < 1.5s on 4G
- Time to Interactive: < 3s
- Lighthouse Mobile Score: ≥ 85
- Images served in WebP; lazy-loaded below the fold

---

## 6. Security Requirements

### 6.1 General Principles
The application must be built with security-first practices appropriate for a consumer-facing mobile web app.

### 6.2 Frontend Security
- **Content Security Policy (CSP):** Strict CSP headers to prevent XSS. Only allow scripts from trusted sources (own domain, CDN with SRI hash).
- **Subresource Integrity (SRI):** All third-party scripts and stylesheets loaded with integrity hashes.
- **No inline scripts or eval():** Enforce via CSP `script-src 'self'`.
- **Input sanitization:** All user inputs (search, chat) must be sanitized before processing. Never render raw HTML from user input.
- **HTTPS only:** Enforce HTTPS via HSTS header with `max-age=31536000; includeSubDomains; preload`.

### 6.3 API Security
- **API keys never exposed in client-side code.** All calls to Anthropic API, Google Maps API, Google Places API, and **Unsplash API** must be proxied through a backend/serverless function.
- **Rate limiting:** Implement per-IP rate limiting on all API proxy endpoints (especially the AI chat endpoint) to prevent abuse. Recommended: 30 requests/minute per IP for search, 10 requests/minute for AI chat.
- **Authentication for "My Trip" save feature:** Use session tokens (JWT, HTTP-only cookies) — never localStorage for sensitive tokens.
- **CORS:** Restrict CORS to own domain only on all API routes.

### 6.4 Third-Party Embeds
- **Maps:** Leaflet + OpenStreetMap default (§7.1). If Google Maps is added later, API keys must use HTTP referrer restriction locked to app domain.
- No unverified third-party scripts (analytics, marketing tools) without security review.

### 6.5 Data Privacy
- No PII collected in Phase 1 beyond analytics (consent-gated). Saved trips and accounts → Phase 2–3 (§11.4).
- GDPR/PDPA compliant cookie consent banner.
- No selling or sharing of user data.

### 6.6 Dependency Management
- `npm audit` run on every build; no high/critical vulnerabilities in production.
- Dependency lock files (`package-lock.json`) committed to source control.
- Regular automated dependency updates via Dependabot or Renovate.

---

## 7. Technical Architecture (Recommended)

### 7.1 Frontend
- **Framework:** Next.js (React) — mobile-first, SSG/ISR for resort pages (fast + SEO), SSR for dynamic content
- **Styling:** Tailwind CSS + custom design tokens (§12.2)
- **Maps:** **Leaflet.js + OpenStreetMap (default for Phase 1–2)** — zero/low API cost. Google Maps JavaScript API is optional for a later phase if POI quality requires it (see R2, §16).
- **State:** Zustand or React Context for trip planner state

### 7.2 Backend / API Layer
- **Serverless functions** (Vercel Edge Functions or AWS Lambda) to proxy:
  - Anthropic Claude API (AI chat + search agent)
  - Google Places API (restaurants, accommodation photos — Phase 2+)
  - Unsplash API (photo metadata, download tracking — key in `UNSPLASH_ACCESS_KEY`, §11.2)
  - Snow report scraping / external APIs (Phase 3+)
- **Resort content pipeline:** Markdown files in `content/resorts/` are **source of truth**; imported at **build time** into static pages (§11.7). No headless CMS in Phase 1–2.

### 7.3 Data Storage
- **Resort data:** Git-managed markdown (`content/resorts/*.md`) → build-time import (§11.7)
- **User saved trips:** Supabase (PostgreSQL + Auth) or Firebase — **Phase 3** when Plan/My Trip ships (§11.4)
- **Cache:** Edge caching (Vercel / Cloudflare) for resort detail pages; TTL 24h for snow condition data (Phase 3+)

### 7.4 AI Integration
- Claude `claude-sonnet-4` model via Anthropic API
- Web search tool enabled for live snow/condition queries
- System prompt scoped to Japan ski trip domain; no general-purpose AI surfaced to user

---

## 8. Content Requirements by Destination

> **Where this content lives (read this first — for implementers and AI agents).**
> The researched resort/area content is **not** in this PRD. It is prepared as separate files so it can be refreshed on a seasonal cadence independent of product changes:
>
> | File | Purpose | Who/what reads it |
> |------|---------|-------------------|
> | `content/content-model.md` | The content schema, volatility tiers, freshness/refresh calendar, and the AI research brief | Editors + the content-generation AI agent |
> | `content/resorts/_TEMPLATE.md` | Blank schema to clone for a new resort | The AI agent when onboarding a resort |
> | `content/resorts/<slug>.md` | The actual prepared content, **one file per resort** (20 files) — YAML frontmatter (structured) + markdown body (narrative) | App build / CMS import + the search/chat agent |
> | `content/site-images.md` | Shared, non-resort image assets (home/gateway-city heroes) with Unsplash attribution | App build (see §11.2, §17.2) |
> | `docs/analytics/tracking-plan.md` | The authoritative analytics event spec (day-one events, properties, naming rules) — instrument against this at launch | The dev/AI agent building the app (see §13) |
>
> **For the next AI agent:** to populate or refresh a resort, open `content/resorts/<slug>.md` (clone `_TEMPLATE.md` if missing) and follow the rules in `content-model.md` §4 (schema) and §5 (research brief). Resort slugs are kebab-case of the name, e.g. `niseko-united.md`, `hakuba-valley.md`, `lotte-arai.md`, `nekoma-alts-bandai.md`. The field list each file must provide is reproduced in §8.1 below.

The following resort data must be researched, written, and present in `content/resorts/<slug>.md` before Phase 1 launch. **All 20 resorts ship in Phase 1** (see §3.1 table). Each resort requires:

- [ ] Hero photograph (licensed)
- [ ] 3–5 highlights with icons
- [ ] 2–4 lowlights with icons
- [ ] Mountain stats (runs, vertical, area, lifts)
- [ ] Typical season dates
- [ ] Average snowfall
- [ ] Nearest village/town description (2–3 paragraphs)
- [ ] Gateway airport + how to get there (2–3 transport options)
- [ ] 3–5 accommodation recommendations
- [ ] 3–5 restaurant recommendations
- [ ] 1–2 gear rental shop listings
- [ ] Airport city guide (for unique gateways: Sapporo, Tokyo, Sendai)

**Content production order (all 20 required for Phase 1 launch; sequence below is research priority):**
1. Niseko United · 2. Hakuba Valley · 3. Furano · 4. Nozawa Onsen · 5. Zao Onsen · 6. Rusutsu · 7. Shiga Kogen · 8. Myoko Kogen · 9. Naeba · 10. GALA Yuzawa · 11. Madarao Kogen · 12. Tomamu · 13. Kiroro · 14. Appi Kogen · 15. Hoshino Resorts Nekoma Mountain · 16. Joetsu Kokusai · 17. Lotte Arai · 18. Tsugaike Kogen · 19. Kandatsu Kogen · 20. Sahoro

---

### 8.1 Per-resort content schema (the data each `content/resorts/<slug>.md` provides)

This is the definitive list of fields the content layer supplies to implementation. It mirrors `content/resorts/_TEMPLATE.md`; the canonical, fully-annotated version lives in `content-model.md` §4. Each field is tagged by **volatility tier**: **T1** = evergreen (written off-season, reviewed yearly), **T2** = seasonal (refreshed Aug–Sept, always label the season), **T3** = live (never stored — fetched at runtime via the AI agent / Places API).

**Identity & classification [T1]**
- `name_en`, `name_jp`, `region`, `prefecture`
- `resort_type` (single mountain / interconnected area / part of valley-pass), `linked_resorts`
- `tagline`, `best_for` (tags: powder, beginners, families, culture, tree runs, nightlife…)

**Access & transport [T1]**
- `gateway_airport`, `airport_code`, `airport_distance_time`, `direct_station`
- `transport_options[]` — `{ mode, duration, approx_cost, note }`
- `recommended_transport`

**Mountain & terrain [T1]**
- `top_elevation_m`, `base_elevation_m`, `vertical_drop_m`, `skiable_area_ha`
- `num_runs`, `longest_run_m`, `terrain_split { beginner, intermediate, advanced }`, `steepest_gradient_deg`
- `lifts`, `terrain_parks`, `night_skiing`, `tree_runs_offpiste_policy`
- `trail_map_url` — link to the **official** slope/piste map (courses colour-coded by difficulty); prefer the resort's own URL
- `trail_map_note` — what the map shows + colour convention (**JP: green = beginner, red = intermediate, black = advanced**)

**Snow & season**
- `avg_annual_snowfall_m` [T1], `snow_quality` [T1], `season_typical` [T1], `best_months` [T1], `peak_avoid` [T1]
- `season_confirmed` [T2 — label the season]
- *current snow/base/open-runs* → [T3, live only — not stored]

**Suitability & accessibility [T1]**
- `family_friendly`, `snowboarder_friendly`, `english_support`, `ski_school`, `on_site_rental`, `ski_in_ski_out`, `adaptive_accessibility`
- Course breakdown narrative for **beginner / intermediate / advanced** (in the markdown body)

**Pricing & passes [T2 — label the season]**
- `lift_pass_adult_day`, `lift_pass_notes`, `multi_resort_pass`

**Village, area & food [T1]**
- `nearest_village`, `nearest_town`, `onsen`, `apres_nightlife`, `local_cuisine`, `nearby_daytrips`

**Curated picks [T1/T2 — verify still open at refresh]**
- `accommodation_picks[]` — `{ name, type, why }` (3–5)
- `food_picks[]` — **top ~5** `{ name, cuisine, why, source }`; triangulate Tabelog + travel guides + Google reviews (not Google alone — many resorts are remote with sparse reviews); include nearby-town spots and say so where on-mountain dining is thin
- `rental_picks[]` — `{ name, tier, why }` (1–2)
> Full live listings/photos/prices/booking links are **not** stored here — they come from the Google Places API at runtime (see §4.3, §3.3–3.5).

**Editorial body (markdown) [T1]**
- `Quick verdict` (2 sentences), `Course breakdown by level`, `Highlights` (3–5), `Watch out for` (2–4), `The area` (1–2 paragraphs)

**Media [T1] — Unsplash, attribution mandatory (§11.2, §17.2)**
- `images.hero[]`, `images.card`, `images.village[]`, `images.cuisine[]` — each entry: `{ photo_id, query, photographer, photographer_url, photo_url, alt }`. Specific restaurant/hotel photos come from Google Places at runtime, not here. Full schema: `content-model.md` §4.11; prep workflow: §5.5. Site-level images: `content/site-images.md`.

**Governance**
- `content_status` (draft / in-review / published), `last_reviewed`, `reviewed_by`, `data_confidence` (high/medium/low), `sources[]`

---

## 9. Out of Scope (V1 / Phase 1)

- Online lift pass or accommodation booking engine (link-out only)
- User accounts, saved trips, and **Plan / My Trip nav tab** (Phase 3 — §11.4)
- Hero video loops on resort pages (static Unsplash imagery only)
- Interactive embedded trail maps (link-out to official map only — §3.2)
- Android/iOS native app (web only for V1)
- Non-Japan ski destinations
- Real-time crowd / wait-time data
- Gear sales or e-commerce
- Headless CMS (Contentful/Sanity) — deferred to Phase 3 editorial workflow (§11.7)

---

## 10. Roadmap & Release Gates

> **Why there are no calendar dates here.** This product is built solo with an AI pair (Claude). Throughput is highly variable and there is no team to coordinate around fixed dates, so time-boxed milestones ("3 months") would be fiction. Instead the roadmap is **dependency-ordered** (what must exist before what) and each release passes **state-based quality gates** that the human owner signs off. **Ship a phase when its gates pass, not when a date arrives.** With the AI doing most of the production, the owner's role is gate-keeper — the gates below are where you stay in control of quality.

### 10.1 Build sequence (dependency-ordered, not time-boxed)

The phases define **order and scope**, so foundations are built before things that depend on them. There is no schedule attached.

**Phase 1 — MVP / foundation**
- Home / discovery page with **all 20** resorts (§3.1)
- Onboarding (Screen 0) + **Find My Resort quiz** (Screen 0b)
- Resort detail pages (highlights, lowlights, data card, trail map link-out)
- Getting There section
- **AI chat — resort Q&A** from floating button (seed-content-first, §4.1; no live snow search yet)
- Bottom nav: **Home · Explore · Info** only (no Plan tab)
- Static map embed where applicable; **Leaflet.js + OpenStreetMap** default
- Markdown resort content → build-time static pages (§11.7)
- Minimal white UI with hero photography + Unsplash attribution
- i18n infrastructure scaffolded (even if only EN ships first)
- **Analytics instrumented from launch** — full day-one taxonomy including quiz events (§13.3, `docs/analytics/tracking-plan.md`)

**Phase 2 — Planning layer** *(depends on Phase 1 resort pages + content)*
- Accommodation and restaurant recommendations + map (Screen 3)
- Gear rental listings
- Village / town explorer
- User accounts with secure authentication (email + OAuth) — auth foundation only; no Plan tab yet
- Thai (TH) and Japanese (JP) language support

**Phase 3 — Plan, freshness & comparison** *(depends on Phases 1–2)*
- **Plan / My Trip tab** added to bottom nav → **Home · Explore · Plan · Info**
- Saved trips linked to user account (server-side, no localStorage)
- Live snow condition search via web search tool (T3 data, §4.3)
- CMS or Notion-backed **editorial workflow** with seasonal review reminders (§11.7)
- Resort comparison feature
- Full trip-planning wizard depth in AI chat

**Phase 4 — Polish & growth (continuous)**
- Performance optimisation and SEO (resort landing pages indexed)
- Additional language support (Simplified Chinese, Korean)
- Monetisation integrations (affiliate booking links, Slopes app partnership)
- Resort direct partnerships for official photography and promotions
- Real-time lift status and crowd data (if resort APIs become available)

### 10.2 Release-readiness gates (human-owned go/no-go)

Each phase passes these checkpoints before it counts as "done." These replace traditional date milestones — they are the moments the human owner reviews and approves AI output.

| Gate | What it means here (solo + AI) | Owner |
|------|--------------------------------|-------|
| **Design review** | UI matches the design spec & language (§5, §12); theme tokens, type, spacing, and responsive/mobile layout verified | You — review AI's output |
| **Implementation checkpoint** | Feature meets its acceptance criteria; security checklist (§6) met; no secrets client-side; clean outbound links only (§11.1). *(This absorbs the traditional "dev handoff" — there is no separate dev team.)* | You + AI self-check |
| **Content freeze** | All in-scope `content/resorts/<slug>.md` files are `content_status: published`, `last_reviewed` stamped, sources verified; season labels correct per §4.3; T3 live fields wired (not hard-coded) | You — editorial |
| **QA sign-off** | Cross-device/responsive pass, accessibility, performance targets, broken-link & embed (trail maps) check | You + AI test pass |
| **Launch** | Deploy, smoke test, monitoring on; post-launch watch for broken external links | You |

---

## 11. Product Decisions (Resolved)

The following questions have been answered by the product owner and are now considered resolved decisions.

### 11.1 Monetisation
**Decision:** No monetisation in Phase 1. The early focus is entirely on producing a genuinely great product that serves users with the best possible information about Japan ski trips. Commercial partnerships (e.g. integrations with apps like Slopes, affiliate links with accommodation booking platforms) will be explored in a later phase once the product has demonstrated quality and user trust.

**Implication for development:** No affiliate tracking code, no booking widgets, no ad infrastructure in V1. All external links are clean outbound links only.

**Cost note:** because there is no revenue to offset running costs in V1, the AI chat feature must stay within the cost guardrails defined in §4.4, and overall AI spend (run-time *and* build-time) follows the discipline in §14. The day-one budget is unknown and depends on launch success — so the design priority is a **bounded worst case** (kill-switch + caps), not a fixed budget.

### 11.2 Photography
**Decision:** Use Unsplash API for all photography in V1. No licensed photography budget is available at this stage. Resort hero images, village photos, and food photography will all be sourced from Unsplash using curated search queries per destination.

**Where images are needed (image slots):**
- **Resort hero carousel** (1–3 images per resort) — the headline shots on each detail page
- **Discovery/list card** thumbnail per resort
- **Village / town character** shots for the area section
- **Illustrative local-cuisine** imagery for the food section (generic dishes — *not* specific named restaurants)
- **Site-level heroes** — home/discovery page and gateway-city imagery (shared, not per-resort)

> Actual photos of **specific restaurants/hotels** come from the **Google Places API at runtime**, not Unsplash — never use an Unsplash stock photo to depict a named business.

**Implication for development:**
- Integrate Unsplash API (free tier) with curated search queries per slot (e.g. "Niseko powder skiing", "Hakuba village winter", "Hokkaido ramen")
- Store selected photo IDs **plus full attribution metadata** (photographer name, profile URL, photo URL, alt text) so images stay consistent and credited — do not re-query Unsplash randomly on each page load
- **Unsplash attribution is legally required on every image** (see §17.2) — render a visible "Photo by [Name] on Unsplash" credit with correct links, and trigger the download endpoint when an image is used
- In a later phase, direct resort partnerships may provide official photography to replace Unsplash content

**Asset preparation (before dev starts):** Images are a **content task prepared ahead of the build**, just like text. The per-slot schema lives in `content/content-model.md` §4.11, and the compliant step-by-step preparation workflow (search → record attribution → trigger download → hotlink → credit) is in §5.5. Per-resort images go in `content/resorts/<slug>.md`; shared site images go in `content/site-images.md`.

**Unsplash API key (environment config):**

| Item | Detail |
|------|--------|
| **Env variable** | `UNSPLASH_ACCESS_KEY` — the public **Access Key** from your Unsplash developer application (not the Secret Key) |
| **Where it lives** | Local `.env` (copy from `.env.example`); injected in production via host secrets (Vercel env vars, etc.). **Never commit the real key** — `.env` is gitignored |
| **UTM source** | `UNSPLASH_UTM_SOURCE=japan_ski_planner` — used in attribution links (§17.2) |
| **How to obtain** | Register at [unsplash.com/developers](https://unsplash.com/developers) → [Your apps](https://unsplash.com/oauth/applications) → create application → copy Access Key |
| **Status** | **Pending developer app approval.** A **screenshot of the site showing Unsplash attribution** is required before Unsplash grants production access. Until approved: demo mode only (**50 requests/hour**); after approval: **1,000 requests/hour** |
| **Security** | Key is **server-side only** — proxy Unsplash calls through a backend/serverless function (§6.3). Never expose in client-side code |
| **What works now (pre-approval)** | One manually selected home hero in `content/site-images.md` (Karsten Winegeart / `L7289nHzVgI`) with full attribution metadata — enough for early UI mockups |
| **Blocked until approval** | Bulk resort image fetch (`scripts/fetch_unsplash_assets.py`), automated download-endpoint tracking at scale, and any runtime flow that re-queries Unsplash on every page load |
| **After approval** | Run `scripts/fetch_unsplash_assets.py` for priority resorts; wire download tracking (`GET /photos/:id/download`) on first image use; store selected `photo_id` + attribution in `images.*` per resort |

### 11.3 Resort Data Accuracy
**Decision:** AI-assisted editorial model. An AI agent will be used to help research and draft resort information, but a human editor reviews and approves before publication.

**Update cadence:** A full data refresh will be conducted approximately 3–4 months before each ski season starts (typically August–September for Japan's December–April season). This refresh covers: lift pass prices, number of open runs, season dates, transportation costs, accommodation pricing tiers, and any new facilities.

**Implication for development:**
- Build/import pipeline must read `last_reviewed`, `content_status`, and `data_confidence` from markdown frontmatter (§8.1)
- Build an admin/Notion view or script that lists resorts where `last_reviewed` is older than the seasonal deadline
- AI search agent can assist editors by pulling latest resort information from official sites during review cycles (Phase 3+ editorial workflow, §11.7)

### 11.4 User Accounts, Saved Trips & Plan Tab
**Decision:** User accounts and the **Plan / My Trip** experience are **not** in Phase 1. Phase 1 bottom nav is **Home · Explore · Info** only; AI chat is reachable via the floating button on resort detail (FR-6).

**Phase 1 approach:** No accounts, no saved trips, no Plan tab. Users discover and Q&A within the session.

**Phase 2 (auth foundation):** Email + OAuth authentication (HTTP-only session cookies). Auth ships without the Plan tab — prepares for Phase 3 saved trips.

**Phase 3 (Plan / My Trip):**
- **Plan tab** added to bottom nav → **Home · Explore · Plan · Info**
- Saved trips stored server-side (Supabase or equivalent), linked to authenticated user
- Exportable itinerary from Plan workspace
- GDPR/PDPA compliant data handling from day one of account launch
- Security audit required before Phase 3 Plan tab ships

### 11.5 Languages
**Decision:** English only for Phase 1. Additional languages will be introduced in Phase 2 or Phase 3 based on user demand and product maturity.

**Language roadmap:**
- Phase 1: English
- Phase 2–3: Thai (TH) — relevant given the strong Thai ski travel market to Japan
- Phase 2–3: Japanese (JP) — for domestic Japanese users and resort staff
- Future: Simplified Chinese (ZH), Korean (KO) — major inbound ski tourism markets

**Implication for development:** Build with i18n architecture from the start (e.g. `next-intl` or `react-i18next`) even if only English strings are loaded in V1. This avoids a painful retrofit when translations are added.

### 11.6 Resort Partnerships
**Decision:** No formal resort partnerships in Phase 1. Partnerships will be pursued organically once the product demonstrates meaningful traffic and user engagement.

**Future vision:** If the product gains traction, direct relationships with resorts could unlock official photography, exclusive promotions, and verified real-time data feeds. Resorts in Hokkaido (Niseko, Furano) and Nagano (Hakuba) have well-established international marketing teams that would be natural first contacts.

**Phase 1 approach:** Ensure all resort information is accurate, respectful, and could be shown to a resort's marketing team without concern. This builds credibility for future outreach.

### 11.7 Content Pipeline (Resolved)
**Decision:** **Markdown in the repo is the source of truth.** Resort content lives in `content/resorts/<slug>.md` with YAML frontmatter (§8.1). The Next.js build **imports markdown at build time** into static/ISR pages — no headless CMS in Phase 1–2.

**Phase 3 editorial workflow:** Introduce a CMS (Contentful, Sanity, or Notion-backed export → markdown) **or** a lightweight admin UI for non-technical seasonal refresh — but the **git markdown files remain canonical** until a deliberate migration decision. Seasonal refresh continues to follow `content-model.md` tiers and §4.3 season currency policy.

**Implication for development:** Do not wire Contentful/Sanity in Phase 1. Build a markdown import layer (e.g. `gray-matter` + content loader) and treat `content_status: published` as the launch gate filter.

## 12. UI/UX Specification

### 12.0 Design References

The visual direction draws from four reference products. Each contributes a different dimension of what this app should feel and behave like. Developers should study all four before building any component. Where references conflict, the hierarchy is: Kinfolk (mood) → Hokuoh Kurashii (mobile structure) → Monocle (editorial rigor) → Airbnb (functional UX patterns).

---

#### Reference 1: Kinfolk Magazine — kinfolk.com
**What it contributes:** Overall mood. Quiet, unhurried, human. Photography breathes. Text is sparse. Every screen should feel like you could slow down and linger.
- Off-white backgrounds (never pure #FFFFFF — always slightly warm)
- Generous vertical whitespace between sections
- Serif headings with occasional italic for pull quotes and standout copy
- No decorative UI chrome — content is the design
- Photography: close-up textures, natural light, human presence without being posed

#### Reference 2: Hokuoh Kurashii / 北欧暮らしの道具店 — hokuohkurashi.com
**What it contributes:** Mobile layout structure and Japanese minimal approach to content. One of the cleanest mobile reading experiences in the Japanese market.
- Full-width imagery with understated text directly below
- Consistent vertical rhythm; sections stack with uniform spacing
- Section labels in small, light-weight uppercase
- Near-monochrome palette with a single warm accent only
- Trust through simplicity: no flashy UI, but everything is findable and readable

#### Reference 3: Monocle Magazine — monocle.com
**What it contributes:** Editorial confidence and information architecture. Monocle never wastes words but always has a point of view.
- Bold, clear section headers that tell you exactly where you are
- Data in clean tables and structured lists, not infographics
- Strong typographic hierarchy throughout
- Geographical tagging (city/region always visible on content)
- Copy that is knowledgeable and direct with warmth, never generic

#### Reference 4: Airbnb — airbnb.com
**What it contributes:** Mobile UX interaction patterns, particularly search, filter, and map browsing — problems this product shares directly.
- Bottom navigation: always visible, no hide-on-scroll behaviour
- **Phase 1 nav:** Home · Explore · Info
- **Phase 3 nav:** Home · Explore · Plan · Info (Plan tab replaces nothing — inserted between Explore and Info)
- Filter chips: horizontal scroll, single-select, instant result update
- Map + list toggle for accommodation browsing
- Card design: image-forward, key info immediately visible
- Map pin tap → bottom sheet slides up with a card preview
- Search: tap → expands with smooth animation, results update live

---

### 12.1 Overall Design Language

**Aesthetic:** Minimal, editorial, premium. Not a booking engine. Not a resort website. Something closer to a trusted magazine that is also deeply useful.

**Mood:** Clean, calm, inspiring. Evokes fresh mountain air, champagne powder, and a lantern-lit Japanese village at dusk. Users should feel guided by a knowledgeable friend, not sold to.

**Tone (copy and AI):** Warm and conversational, like a well-travelled friend giving honest advice to someone planning their Japan ski trip. Truthful about lowlights. Never promotional. Respectful of Japanese culture throughout.

### 12.2 Color System — Theme Variants

The app ships with one active theme (Default). The others are documented here so a developer can implement theme switching in a later phase without refactoring the token system. All themes use the same CSS variable names; only the values change.

**How to implement:** Define all colors as CSS custom properties on `:root`. To switch themes, swap the values on a `data-theme` attribute on `<html>`. This means zero component changes to switch themes.

```css
/* Example structure */
:root[data-theme="default"] { --color-bg: #F7F5F2; ... }
:root[data-theme="monocle"] { --color-bg: #FFFFFF; ... }
```

---

#### Theme A: Default (Kinfolk-inspired) — **Active for V1**

| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg` | `#F7F5F2` | Warm off-white page background |
| `--color-surface` | `#FFFFFF` | Cards, modals, overlays |
| `--color-text-primary` | `#1C1A17` | Headings, body text |
| `--color-text-secondary` | `#6E6A63` | Labels, captions, secondary info |
| `--color-accent` | `#2D6BE4` | CTAs, active states, highlight badges |
| `--color-accent-warm` | `#C4873A` | Lowlight badges, warm secondary accents |
| `--color-divider` | `#E8E5E0` | Borders, section dividers |
| `--color-highlight-bg` | `#EEF3FF` | Highlight pill background |
| `--color-lowlight-bg` | `#FFF4E3` | Lowlight pill background |
| `--color-nav-bg` | `#FFFFFF` | Bottom nav, sticky header background |

Typography pairing: **Cormorant Garamond** (display/headings) + **DM Sans** (body/UI)

---

#### Theme B: Hokuoh Kurashii (Warm Minimal Japanese)

| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg` | `#FAFAF8` | Very pale warm white |
| `--color-surface` | `#FFFFFF` | Cards |
| `--color-text-primary` | `#222222` | Near-black |
| `--color-text-secondary` | `#888888` | Light grey |
| `--color-accent` | `#B5956A` | Warm tan/ochre — the Hokuoh signature |
| `--color-accent-warm` | `#D4A574` | Lighter warm tan |
| `--color-divider` | `#EEEEEE` | Very light grey |
| `--color-highlight-bg` | `#F5EFE6` | Warm beige |
| `--color-lowlight-bg` | `#F0F0F0` | Neutral grey |
| `--color-nav-bg` | `#FAFAF8` | Matches background |

Typography pairing: **Noto Serif JP** (headings, bilingual-ready) + **Noto Sans JP** (body)

---

#### Theme C: Monocle (Clean Editorial)

| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg` | `#FFFFFF` | Pure white |
| `--color-surface` | `#F4F4F4` | Card backgrounds |
| `--color-text-primary` | `#000000` | Pure black headings |
| `--color-text-secondary` | `#555555` | Mid grey |
| `--color-accent` | `#CC0000` | Monocle red — CTAs, tags |
| `--color-accent-warm` | `#E8A000` | Amber for secondary |
| `--color-divider` | `#DDDDDD` | Light grey borders |
| `--color-highlight-bg` | `#FFF0F0` | Light red tint |
| `--color-lowlight-bg` | `#F4F4F4` | Neutral surface |
| `--color-nav-bg` | `#FFFFFF` | White |

Typography pairing: **Helvetica Neue** or **Inter** (all weights) — no serif in this theme

---

#### Theme D: Airbnb (Functional Warm)

| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg` | `#FFFFFF` | White |
| `--color-surface` | `#FFFFFF` | Cards |
| `--color-text-primary` | `#222222` | Near-black |
| `--color-text-secondary` | `#717171` | Airbnb grey |
| `--color-accent` | `#FF385C` | Airbnb coral/red |
| `--color-accent-warm` | `#FF6B6B` | Lighter coral |
| `--color-divider` | `#DDDDDD` | Standard grey |
| `--color-highlight-bg` | `#FFF0F2` | Coral tint |
| `--color-lowlight-bg` | `#F7F7F7` | Light grey |
| `--color-nav-bg` | `#FFFFFF` | White |

Typography pairing: **Cereal** (Airbnb's own font, not freely available) → substitute with **Plus Jakarta Sans** which shares similar geometric character

### 12.3 Typography — Theme Variants

Each theme has its own type pairing. The token names remain constant; only the font values change.

#### Default Theme Typography (Active for V1)

| Token / Role | Font | Weight | Size (mobile) | Notes |
|---|---|---|---|---|
| `--font-display` | Cormorant Garamond | 600 | 36–48px | Hero titles, resort names |
| `--font-display` italic | Cormorant Garamond | 400 italic | 22–28px | Pull quotes, verdict summaries |
| Section heading (H2) | Cormorant Garamond | 500 | 26px | Page sections |
| Card title (H3) | DM Sans | 600 | 18px | Resort card, accommodation card |
| Body text | DM Sans | 400 | 15px | Descriptions, body copy |
| Caption / Label | DM Sans | 400 | 12px | Attribution, dates, secondary info |
| Badge / Tag | DM Sans | 500 | 11px | Uppercase, +0.08em letter-spacing |
| Navigation | DM Sans | 500 | 11px | Bottom nav labels |

Line height: 1.6 for body, 1.15 for display headings. Letter-spacing: +0.05em for uppercase labels and tags. Load from Google Fonts: `Cormorant+Garamond:ital,wght@0,500;0,600;1,400` and `DM+Sans:wght@400;500;600`.

#### Hokuoh Theme Typography
- Headings: Noto Serif JP, weight 600 — bilingual-ready, clean Japanese serif
- Body: Noto Sans JP, weight 400 — excellent readability for both English and Japanese

#### Monocle Theme Typography
- All text: Inter (or Helvetica Neue if available) — tight tracking on headings, no serif

#### Airbnb Theme Typography
- All text: Plus Jakarta Sans — geometric, friendly, highly legible at small sizes

### 12.4 Spacing & Layout — Theme Variants

Spacing is also tokenised so themes can breathe differently. Default (Kinfolk) uses the most generous spacing.

| Token | Default (Kinfolk) | Hokuoh | Monocle | Airbnb |
|---|---|---|---|---|
| `--space-screen-x` | 20px | 16px | 24px | 24px |
| `--space-card-gap` | 20px | 16px | 16px | 16px |
| `--space-section-gap` | 48px | 40px | 32px | 32px |
| `--space-card-padding` | 16px | 14px | 16px | 16px |
| `--radius-card` | 14px | 8px | 4px | 12px |
| `--radius-button` | 10px | 4px | 2px | 8px |
| `--radius-badge` | 99px | 99px | 2px | 99px |
| Shadow style | None / `0 2px 10px rgba(0,0,0,0.05)` | None | `0 1px 4px rgba(0,0,0,0.10)` | `0 1px 8px rgba(0,0,0,0.08)` |
| Divider style | `1px solid var(--color-divider)` | `1px solid #EEEEEE` | `1px solid #DDDDDD` | `1px solid #DDDDDD` |

**Base grid:** 8px unit. All spacing values are multiples of 8. No hard shadows in Default or Hokuoh themes — borders only.

### 12.5 Screen-by-Screen Specifications

All screens are designed for 375–430px viewport width (iPhone SE → iPhone Pro Max). Bottom navigation is always visible — it does not hide on scroll.

**Navigation by phase:**
| Phase | Bottom nav tabs |
|-------|-----------------|
| **1** | Home · Explore · Info |
| **3+** | Home · Explore · Plan · Info |

**Explore tab (Phase 1):** Resort discovery with filters and quiz entry point — shares the same resort list as Home but opens with filter chips and "Find my resort" CTA prominent. Implementation may reuse Home with a different default scroll/focus; not a separate data source.

**Screen index:**

| Screen | Name | Phase |
|--------|------|-------|
| 0 | Onboarding | 1 |
| 0b | Find My Resort (quiz) | 1 |
| 1 | Home / Resort Discovery | 1 |
| 2 | Resort Detail | 1 *(Phase 2 sections marked inline)* |
| 3 | Accommodation & Map | 2 |
| 4 | Getting There | 1 |
| 5 | AI Chat | 1 *(wizard depth → Phase 3)* |
| 6 | Info | 1 |

---

#### Screen 0: Onboarding (First-Time Only) · **Phase 1** · FR-5

Shown once on first visit, not shown again. No heavy animation. Illustrations are minimal and used only as gentle visual support — icons and simple line art only, never large decorative illustrations that take up significant space.

**Layout:**
- 3–4 swipeable onboarding cards, each taking the full screen
- Card 1: App name + tagline ("Your guide to Japan's best snow") + a single small icon (mountain or snowflake line art)
- Card 2: "Find your resort" — brief sentence about the resort discovery feature, single relevant icon
- Card 3: "Plan the whole trip" — accommodation, food, getting there, brief sentence + icon
- Card 4 (optional): "Ask anything" — AI chat feature, brief sentence + icon
- Progress dots at bottom; skip button top-right
- "Get started" CTA on final card
- Background: `--color-bg` warm off-white; no photographs on onboarding cards (keeps loading fast and focus on message)

**Tone of copy:** Warm and welcoming. "Winter in Japan is unlike anywhere else. Let's help you find your perfect season." — not "Sign up to access features."

---

#### Screen 0b: Find My Resort (Resort Discovery Shortcut) · **Phase 1** · FR-4

Accessible from onboarding final card and from Home screen. A sliding card quiz — not a form or questionnaire. Each question is a single full-width card; user swipes or taps to select their answer and the next card slides in from the right.

**Quiz modes (Phase 1):** Before the first question, user toggles **Serious** (default) or **Cosmic**. Both variants present four questions with wide full-width answer buttons. Display copy differs; **scoring enums and resort results are identical** for equivalent answers. Full question sets, enum mapping, and implementation notes: [`features/quiz/brief.md`](../../features/quiz/brief.md).

**Serious mode — questions (canonical, 4 max for V1):**

| Card | Question | Answer options (tappable, wide buttons) |
|------|----------|----------------------------------------|
| 1 | What's your level on the mountain? | Just starting out · Getting confident · I charge hard · It varies in my group |
| 2 | What matters most to you? | The powder & terrain · Village & culture · Family-friendly · All of the above |
| 3 | Where does your flight land? | Hokkaido (Sapporo) · Honshu (Tokyo/Nagano) · Tohoku (Sendai) · Not sure yet |
| 4 | When are you thinking of going? | Early season (Dec–Jan) · Peak powder (Feb) · Late season (Mar–Apr) · Still planning |

**Cosmic mode — questions:** Playful, oblique copy (e.g. morning park invite, ideal evening, winter postcard, bookstore travel book). Maps 1:1 to Serious enums — see [`features/quiz/brief.md`](../../features/quiz/brief.md) Variant B.

**Results reveal + podium (required):**
- **Serious:** header *"Based on your answers, here are resorts we'd recommend for you."* → **podium top 3**
- **Cosmic:** ~800ms interstitial *"Consulting the powder oracle…"* → header *"The stars have spoken. Here are your resorts:"* → subline *"Ranked by cosmic vibes and extremely real resort data."* → **podium top 3**
- **Podium:** #1 featured (accent border, rank **"1"** + label, taller hero, match reason) + #2/#3 minimal (48px thumb, name, region) + *Browse all resorts →* → Explore
- `score.ts` ranks all 20 internally; only top 3 display on quiz results. Home/Explore unchanged.
- Full spec: [`features/quiz/brief.md`](../../features/quiz/brief.md) § Results reveal + § Results podium. i18n keys in `web/messages/en.json`.

Answer options should be wide (full-width buttons, not small chips) to be easy to tap on mobile.

In Phase 3 when user accounts exist, quiz preferences are saved and refined over time through browsing and trip-building behaviour.

---

#### Screen 1: Home / Resort Discovery · **Phase 1** · FR-1

**Layout:**
- Sticky top bar: app logo (left) + search icon (right). Transparent over hero; transitions to white with `1px border-bottom` on scroll past hero.
- **Hero carousel (full-bleed, 65vh):** Auto-advances with a slow crossfade transition (3–4 second hold per image, 1 second fade). No slide animation — crossfade only, calm not busy. The carousel cycles through these curated scene types (developer to source matching Unsplash images):
  1. A lone skier or snowboarder centred in frame, deep powder spraying around them — wide open mountain, blue sky or flat light
  2. A small Japanese ski village at dusk or night — lanterns, snow on rooftops, warm window light. Village examples: Nozawa Onsen village, Akakura Onsen, Hirafu village (Niseko), Ningle Terrace (Furano), Unkai Terrace (Tomamu) at night
  3. A long wooden ski locker or equipment hang area — skis and snowboards racked in a row after a day's riding, steam or cold air visible, nobody in frame
  4. A group of 3–4 people enjoying yakiniku or hot pot inside a rustic local restaurant — warm interior light, casual and joyful
  5. (1 in 5 — food slot) A close-up of a bowl of ramen, a steaming onsen tamago, or a plate of Hokkaido seafood in a simple, beautiful setting
- White gradient overlay at hero bottom for text legibility. App tagline in Cormorant Garamond 600 white, centred or bottom-left.
- Horizontal scrollable filter chips below hero: **All · Hokkaido · Nagano · Niigata · Tohoku · Beginner Friendly · Deep Powder · Family**. Single-select; tapping a chip filters the resort list instantly without page reload.
- **Resort list (below chips):** Single-column vertical scroll. **All 20 resorts** available — initial render may show 10 with "Show more resorts →" loading the remainder inline (performance optimisation allowed; all 20 must be reachable without leaving Home).
- **Load more (optional UX):** If paginating for performance, ensure all 20 are reachable in ≤2 taps from Home.
- **Iteration 2 (post-launch, after user testing):** Test a featured large card (first resort, tall aspect ratio) followed by smaller standard cards below. Do not build this in Phase 1 — validate first whether users engage more with the uniform list or a featured top card.
- Bottom: always-visible sticky bottom nav bar — **Home · Explore · Info** *(Phase 1). Phase 3 adds Plan → **Home · Explore · Plan · Info**.*

**Interactions:**
- Filter chip tap: instant list filter, no page navigation
- Card tap: shared-element transition — image expands from card position to fill Resort Detail hero
- Search icon: expands search bar inline (slides down from top bar) with smooth animation; results update live as user types

---

#### Screen 2: Resort Detail · **Phase 1** · FR-2

**Layout:**
- Full-bleed hero image (55vh) — same Unsplash image as the resort card, so the shared-element transition is seamless
- Resort name overlaid bottom-left of hero in Cormorant Garamond 600 white (large, confident). Region tag as small white pill above the name.
- Back arrow top-left: translucent dark pill button overlaid on image
- **Quick Verdict:** Positioned as a 2-sentence italic pull quote in Cormorant Garamond overlaid directly on the lower portion of the hero image, separated from the resort name by a thin white line. White text with a subtle dark gradient ensuring legibility. Example: *"Niseko is the gold standard for powder in Japan. World-class snowfall, a buzzing international village, and terrain for every level."* This keeps the screen photography-dominant — the verdict rides the image rather than interrupting the white content below.
- Scrollable white content area begins below hero
- **Section order (Phase 1):** Season snapshot → Highlights & Lowlights → Terrain & Mountain Data → Trail map link → Getting There
- **Section order (Phase 2 adds):** → Accommodation → Restaurants → Rental Shops → Nearby Villages & Towns
- Each section: thin `--color-divider` top rule + small uppercase DM Sans section label

**Highlights & Lowlights layout:**
- Two columns, side by side, each taking 50% of the screen width
- Left column header: "Highlights" (accent blue label)
- Right column header: "Watch out for" (amber label)
- Items stack vertically within each column — icon (16px) + short label text (13px DM Sans). Each item is its own row.
- Background tints: highlight items on `--color-highlight-bg`, lowlight items on `--color-lowlight-bg`
- This keeps the comparison scannable without horizontal scrolling

- Floating "Plan with AI →" button pinned just above bottom nav (accent blue, full-width, 52px height) — **Phase 1**; opens Screen 5

---

#### Screen 3: Accommodation & Map · **Phase 2**

**Layout:**
- Header: "Where to stay — [Resort Name]"
- Pill toggle (centred): **List · Map** — smooth transition between views
- Filter chips: **All · Ski-in/Ski-out · Ryokan · Budget · Luxury**
- **List view:** Vertical scroll of accommodation cards. Each card: left-side image (90px × 90px, rounded), right side: name, type tag, price tier (¥/¥¥/¥¥¥), distance to lift, ski-in/ski-out badge if applicable.
- **Map view:** Full-height interactive map (**Leaflet.js + OpenStreetMap** default). All pins visible at once: accommodation (blue pin), restaurants (orange pin), rental shops (green pin). Tapping any pin: bottom sheet slides up (Airbnb-style) with a mini card showing name, type, and key info. Full card available by tapping the mini card.
- **Phase 4–5 addition:** Users can add their own pins — paid accommodation, preferred restaurant, chosen rental shop. This requires user accounts and is out of scope for V1–3.

**Map style:** Monochrome light (equivalent to Google Maps' "Silver" style or Mapbox Light). No terrain elevation shading. Standard roads and place labels only. Keep the map as a wayfinding tool, not a design feature.

---

#### Screen 4: Getting There · **Phase 1** · FR-3

- Clean scrollable page, no tabs
- Transport options: each option is a tappable row — icon (car / train / bus) + method + estimated duration + estimated cost range + external booking link chip
- Gateway airport city section: 3–5 experiences each shown as a small card (Unsplash image + name + 1-line description). Collapsible "See more" if more than 3.
- Practical tips at bottom: collapsible FAQ rows (tap to expand) covering: cash vs card, onsen etiquette, JR Pass, IC Card, language basics

---

#### Screen 5: AI Chat — Plan My Trip · **Phase 1** · FR-6

- Triggered by the "Plan with AI →" floating button on Resort Detail (**Phase 1**). **Phase 3** also enables entry from the Plan tab once it ships.
- Full-screen overlay slides up from bottom
- Close (×) button top-right returns to previous screen
- **Phase 1 scope:** resort Q&A and light trip guidance from seed content (§4.1). Live snow search and full multi-day wizard → **Phase 3**.
- **AI first message (warm, conversational):** "Hey! I'm here to help you put together your Japan ski trip. Tell me a bit about yourself — your level on the mountain, who you're going with, and roughly when you're thinking of going. We can go from there."
- Chat bubbles: user messages right-aligned (accent blue background, white text), AI messages left-aligned (white card, `--color-divider` border)
- AI responses can embed inline mini resort cards, accommodation cards, or transport summaries as rich components within the chat bubble
- Input bar pinned to bottom, keyboard-aware (lifts above keyboard when open)
- AI typing indicator: three dots animated

---

#### Screen 6: Info · **Phase 1** · FR-7

- Accessible from **Info** tab in bottom nav
- **About:** 2–3 sentences on product purpose (Japan ski trip planning, honest editorial tone)
- **Legal:** links to Privacy Policy and Terms of Service (§17.1) — required before Ring 2 public sharing (§15)
- **Contact:** email for feedback and data-rights requests (PDPA/GDPR)
- **Version:** app version string at bottom
- Minimal layout — no photography; fast load

---

### 12.6 Component Library

| Component | Description |
|-----------|-------------|
| `ResortCard` | Full-width image + resort name + region tag + 2–3 badge pills + chevron |
| `ResortCardFeatured` | Taller variant for Iteration 2 home screen (post-launch test) |
| `BadgePill` | Icon + label; variants: highlight (blue tint), lowlight (amber tint), neutral (grey) |
| `HighlightLowlightRow` | Side-by-side 2-column layout, highlights left / lowlights right, items stack vertically |
| `QuickVerdict` | Italic serif pull quote, overlaid on hero image with gradient backing |
| `StatRow` | Horizontal row of 3–4 quick stat chips (runs, vertical, snowfall, season) |
| `AccommodationCard` | Thumbnail left + info right; ski-in/ski-out flag prominent |
| `RestaurantCard` | Thumbnail + name + cuisine + rating + reservation badge |
| `TransportRow` | Icon + method + duration + cost + external link |
| `FilterChip` | Active/inactive state; single-select; instant filter behaviour |
| `MapPin` | 3 variants: accommodation (blue), restaurant (orange), rental (green) |
| `MapBottomSheet` | Slide-up panel on pin tap; mini card with expand affordance |
| `ChatBubble` | User and AI variants; AI supports embedded rich card content |
| `OnboardingCard` | Full-screen swipeable card with icon, title, body, progress dots |
| `ResortQuizCard` | Full-screen sliding quiz card; wide tappable answer buttons |
| `SectionHeader` | Small uppercase label + optional "See all →" link |
| `HeroCarousel` | Full-bleed crossfade carousel; auto-advance; no visible controls |
| `BottomNav` | 4-tab always-visible navigation bar; active state with accent underline or dot |

---

### 12.7 Resort Comparison — Mobile Approach (Future Phase)

Side-by-side columns do not work on mobile without horizontal scrolling, which is a poor UX pattern. For a future phase, the recommended approach is a **stacked attribute comparison** pattern:

- User selects 2–3 resorts to compare (via checkbox on resort cards or a "Compare" mode)
- Comparison screen shows attributes as rows, each row spanning full width
- Each row: attribute name (e.g. "Snow quality") as a left label, then 2–3 resort values stacked or arranged as small value chips side by side within the row
- Because values are short (e.g. "Champagne powder" / "Heavy wet snow" / "Varied") they fit 2–3 across a row without scrolling
- Resort names appear as column headers; if 3 resorts, they abbreviate (e.g. "Niseko / Hakuba / Zao")
- Key attributes: Snow quality, Terrain variety, Beginner friendly (Y/N), Ski-in ski-out (Y/N), Lift pass cost, Distance from airport, Season length, Village/town quality
- This is deferred to Phase 3 or later. Do not build in V1–2.

---

### 12.8 Motion & Animation

**Principle:** Motion should feel calm and purposeful, not playful or energetic. No bounce physics. No particle effects. No entrance animations for decorative purposes.

- **Hero carousel:** Crossfade only (opacity transition). 3.5 seconds per image, 1 second crossfade. No ken-burns/zoom effect.
- **Page transition (card → detail):** Shared element transition — resort card image expands from its card position to fill the detail hero. 280ms ease-out.
- **Filter chip select:** Subtle scale 0.97 → 1.0 on tap. 150ms. No bounce.
- **Cards on scroll (list):** Fade-in + translateY(10px → 0). Staggered 60ms between cards. Only on first render, not on re-scroll.
- **Bottom sheet (map pin):** Slide up from bottom. 260ms ease-out. Backdrop dims to `rgba(0,0,0,0.3)`.
- **Onboarding card swipe:** Standard horizontal slide. No spring overshoot.
- **Quiz card advance:** Horizontal slide from right on answer tap. 200ms.
- **All transitions max:** 300ms. Nothing should feel slow.
- **No animations on:** Illustrations, icons, or any element that doesn't require user attention. Illustrations are static.

---

### 12.9 Illustrations

Illustrations are permitted only in specific, functional contexts. They are not decorative. They should never be the dominant element of a screen.

**Permitted uses:**
- Onboarding cards: one small line-art icon per card (e.g. mountain outline, snowflake, map pin). Maximum 80×80px. Single colour (accent or charcoal).
- Empty states: a simple line illustration when a filter returns no results. Subtle, not whimsical.
- Bottom navigation icons: minimal line icons (Lucide or Phosphor icon set)
- Section icons alongside highlight/lowlight labels

**Not permitted:**
- Full-screen or large decorative illustrations
- Illustrated characters or avatars
- Illustration as a substitute for photography
- Animated illustrations (Lottie files etc.)

---

### 12.10 Empty & Loading States

- Resort card skeleton: grey shimmer rectangle matching full card dimensions. 3–4 shown on initial load.
- Map loading: subtle spinner centred on map tile area; tiles load progressively (standard map behaviour)
- AI chat loading: three-dot typing indicator (standard pattern)
- Empty filter result: small line-art illustration (mountain outline) + "No resorts match — try a different filter"
- Image load error: `--color-surface` placeholder with a small grey mountain icon centred

---

### 12.11 Accessibility

- Minimum contrast ratio 4.5:1 for all body text; 3:1 for large display text
- All interactive elements minimum 44×44px tap target
- Screen reader labels (`aria-label`) on all icon-only buttons and navigation items
- Focus ring: 2px solid `--color-accent` with 2px offset — visible on keyboard navigation
- Images: descriptive alt text required (e.g. "Powder skier at Niseko with snow spray", not "ski image")
- Hero carousel: pause on focus/hover; reduced-motion media query disables crossfade transition and shows static image
- Quiz cards: keyboard-navigable with arrow keys; answer buttons have clear focus states

---

## 13. Analytics, Measurement & Experimentation

> **Where the tracking plan lives (read this first — for implementers and AI agents).** The authoritative, always-current list of events, properties, and naming rules is **`docs/analytics/tracking-plan.md`**, not this PRD. §13 below is the *strategy and rationale*; `docs/analytics/tracking-plan.md` is the *implementation spec you instrument against*. **For the AI agent building Phase 1:** open `docs/analytics/tracking-plan.md`, wire up every event listed there, verify them in PostHog's live-events view, and update that file in the same change whenever you add or alter an event.

The Phase 2 personalisation idea — tailoring suggestions from a user's browsing and search behaviour (§3 quiz note, "preferences refined over time through browsing and trip-building behaviour") — only works if we **capture clean behavioural data from day one**. You cannot retro-fit history: events not tracked at launch are gone forever. This section defines what to instrument on launch, the rules for instrumenting future features, and how experimentation works in later phases.

### 13.1 Measurement principles (the rules)

1. **Privacy-first and lawful by default.** PDPA (Thailand) and GDPR apply. Default to **cookieless, anonymous** measurement; richer/identified tracking only **after consent** (ties to §6 consent banner). Never put PII in event properties.
2. **Measure before you build.** No feature ships without its events defined first. The tracking plan is part of the acceptance criteria checked at the **implementation checkpoint** gate (§10.2).
3. **Every event maps to a decision.** If we can't name the decision a metric informs, we don't track it. No vanity metrics.
4. **One tracking plan, one source of truth.** All events, properties, and their owners live in a single living document (`docs/analytics/tracking-plan.md`). The dev/AI agent updates it in the same change that adds the event.
5. **Consistent naming.** `object_action`, snake_case, past tense (e.g. `resort_card_tapped`). Properties snake_case. No free-form ad-hoc events.
6. **Anonymous → identified continuity.** Use a stable anonymous ID pre-login; on account creation, alias it to the user ID so pre-signup behaviour can power personalisation (Phase 2) without re-collecting consent improperly.

### 13.2 Tooling recommendation (for the dev team)

| Option | Verdict |
|--------|---------|
| **GA4** | **Not recommended as primary.** Free and ubiquitous, but weak for product event funnels, sampling on large queries, awkward PDPA/GDPR consent story, and data ownership concerns. |
| **Plausible** | **Recommended as a lightweight companion.** Cookieless, no consent banner needed, EU-hosted, great for traffic/SEO/top-of-funnel. Lacks funnels, cohorts, flags. |
| **PostHog** | **Recommended as the primary product-analytics backbone.** One tool covers events, funnels, cohorts, session replay, **feature flags, and A/B experiments** — ideal for a solo + AI team that doesn't want to wire up multiple systems. Use **EU Cloud** (or self-host) for PDPA/GDPR; supports cookieless/memory-persistence mode pre-consent. |

**Recommended stack:** **PostHog (EU Cloud)** as the product-analytics + experimentation backbone, with **Plausible** as a privacy-clean marketing/traffic layer. Skip GA4. This keeps experimentation (§13.5) in the *same* tool that already holds the event data — no extra integration.

### 13.3 Day-one event taxonomy

Instrument these from launch (Phase 1), even though heavy *use* of the data starts Phase 2. Page/screen views are captured automatically; the rest are explicit. Always attach common properties (`resort_slug`, `locale`, `device_type`, `is_logged_in`, `consent_state`) where relevant.

| Event | Key properties | Why it matters |
|-------|----------------|----------------|
| `resort_card_tapped` | `resort_slug`, `position`, `list_context` (home / filtered / quiz / comparison), `filters_active` | Core discovery signal; feeds personalisation & resort popularity |
| `filter_applied` | `filter_type`, `filter_value`, `results_count` | Which attributes users actually plan around (powder, beginner, budget…) |
| `load_more_clicked` | `list_context`, `current_count` | Whether the default list length is right; depth of browsing |
| `resort_detail_viewed` | `resort_slug`, `source` (card / search / chat / deep-link) | Primary success metric (§2: ≥70% engage detail pages) |
| `trail_map_opened` | `resort_slug` | Demand for terrain detail; validates the trail-map content effort (§8.1) |
| `accommodation_map_interacted` | `resort_slug`, `action` (pan / zoom / pin_tap) | §2 metric: ≥40% interact with accommodation map |
| `map_pin_tapped` | `resort_slug`, `poi_type` (accommodation / food / rental), `poi_name` | Which POIs draw interest; informs curation |
| `external_link_clicked` | `link_type` (accommodation / rental / official / trail_map / food), `destination_domain`, `resort_slug` | Conversion proxy pre-monetisation; high-value intent signal |
| `ai_chat_opened` | `entry_point` (floating button / CTA / empty-state) | Demand for the AI agent; entry-point effectiveness |
| `ai_chat_message_sent` | `message_index`, `intent` (if classifiable) | Phase 3; query themes feed content gaps & personalisation |
| `quiz_started` / `quiz_completed` | `entry_point`, `quiz_mode` (`serious`/`cosmic`); enums: `level`, `priority`, `region`, `timing`, `result_count` | Onboarding funnel; preference seed for personalisation |
| `search_performed` | `query_text` (only if consent + no PII), `results_count` | Content-gap discovery; powers tailored suggestions |
| `getting_there_viewed` | `resort_slug` | Validates the transport content investment |
| `trip_saved` | `resort_slug` | Phase 2 (requires account); strongest intent signal |

> Capture `query_text` and AI message content only with consent and with PII scrubbing; these power the Phase 2 "tailored suggestions" feature, so getting the schema right on day one matters.

### 13.4 Baseline dashboard (launch)

A single overview board the owner checks regularly, organised top-to-bottom:

1. **North-star & §2 goals** — % users reaching a resort detail page (target ≥70%), % interacting with accommodation map (target ≥40%, Phase 2+), satisfaction score (≥4.2/5, from micro-survey).
2. **Guardrails (§2)** — home bounce rate (≤50%), filter-zero-results rate (≤10%), external-link CTR trend.
2. **Acquisition** (Plausible) — visitors, sources, top landing pages, device split, locale (EN/TH/JP).
3. **Core discovery funnel** — `home → resort_card_tapped → resort_detail_viewed → accommodation_map_interacted → external_link_clicked`, with drop-off at each step.
4. **Feature adoption** — filter usage rate & top filters, load-more rate, AI chat open rate, quiz start→complete rate.
5. **Content insights** — most-viewed resorts, most-used filters, external-link CTR by `link_type`, top search/AI query themes (content-gap radar).
6. **Tech health** — Core Web Vitals (LCP/INP/CLS), JS error rate, map/API failure rate.

### 13.5 Rules for adding analytics to future features

When any new feature is built (by you + AI), it must, before the implementation-checkpoint gate (§10.2):

1. **Define its events first** in `docs/analytics/tracking-plan.md` (name, trigger, properties, owner) following the §13.1 naming convention.
2. **State its success metric** and where it appears on the dashboard (§13.4) — what would make this feature a success or a failure?
3. **Pass a privacy check** — no PII in properties, respects `consent_state`, anonymous-safe.
4. **Ship instrumented** — the events are wired and verified (PostHog live-events view) as part of "done," not added later.
5. **Avoid duplication** — reuse existing events/properties before inventing new ones.

### 13.6 Experimentation (Phase 3–4)

Once there's enough traffic, move from "ship and hope" to **evidence-based iteration**, using PostHog's built-in feature flags + experiments (no new tool needed).

**Start with feature flags (safe rollout):** even before formal A/B tests, gate new features behind flags so they can be rolled out gradually and killed instantly if something breaks. This is valuable from Phase 1.

**What to experiment with (Phase 3–4):**
- **UI:** resort-card layout, CTA copy/placement, filter vs. quiz as the primary discovery entry point, comparison-view patterns.
- **Content:** default resort ordering, "quick verdict" phrasing, photo selection, how season labels are surfaced (§4.3).
- **Features:** AI chat entry points, onboarding-quiz length, personalised vs. generic home ordering (Phase 2 payoff).
- **AI:** system-prompt / response-style variations, measured on chat engagement and downstream detail views.

**Experiment discipline:**
1. Write a one-line **hypothesis** + the **primary metric** + **guardrail metrics** (e.g. don't let bounce rate rise) before starting.
2. Decide **sample size / minimum detectable effect** up front; **don't peek and stop early** — let it reach significance.
3. Change **one thing** per experiment so the result is attributable.
4. Record outcome (ship / kill / iterate) in the tracking plan so learnings compound.

> **Solo + AI reality check:** experiments need traffic to be valid. Until volume supports statistical significance, lean on **feature flags + session replay + qualitative signals** rather than under-powered A/B tests. Run formal experiments only on high-traffic surfaces (home, resort cards).

---

## 14. AI Cost Optimisation — Using Tokens Wisely

There are **two separate AI budgets** for this product, and both need discipline because there's no revenue cushion in V1:

1. **Run-time** — Claude calls the *product* makes to serve users (the chat agent). Hard controls are in §4.4; the principles below explain *how to use tokens wisely* within them.
2. **Build-time** — Claude/Cursor calls *you* make while building and maintaining the site. This is a real, ongoing cost line that's easy to ignore.

### 14.1 Run-time principles (the product's AI spend)

The golden rule: **the cheapest token is the one you never send.** In priority order:

1. **Answer from free content first.** ~90% of questions map to evergreen seed content (§8.1). Use retrieval / a cheap model to answer those; reserve premium-model calls for real synthesis (trip planning).
2. **Cache aggressively.** Prompt-cache the system prompt + resort context (§4.4.3). Edge-cache common Q&A. Share one snow-search result across all users via TTL cache (§4.4.2) — never search per user.
3. **Right-size the model.** Route simple/classification work to a small model (Haiku), complex planning to Sonnet. Don't pay Sonnet prices for "is this a snow question?"
4. **Send less context.** Retrieve only relevant fields; summarise old conversation turns instead of resending them.
5. **Cap outputs.** Tight `max_tokens` everywhere — concise answers are cheaper *and* better UX on mobile.
6. **Make the worst case bounded.** Per-IP caps + a global daily kill-switch (§4.4.1) so a traffic spike or abuse can't produce a surprise bill.
7. **Watch the meters.** Track tokens/cost per session as an analytics dimension (§13) so cost regressions surface early.

### 14.2 Build-time principles (developing & maintaining the site with AI)

Building solo with Claude is itself a metered cost. To get maximum leverage per token:

- **Reuse durable context instead of re-explaining.** The PRD, `content-model.md`, and `docs/analytics/tracking-plan.md` exist so you (and the agent) state intent once and reference it — point the agent at the file rather than re-pasting requirements each session.
- **Scope the context tightly.** Attach the *specific* files a task needs (the §8 file-map helps), not the whole repo. Smaller, focused context = cheaper and more accurate output.
- **Plan before generating.** Planning is cheap; regenerating wrong code after a vague prompt is expensive. Agree the approach, then implement.
- **Right model for the job.** Use a fast/cheap model for boilerplate, scaffolding, renaming, and search; reserve the strongest model for architecture and gnarly debugging.
- **Work in focused tasks and reset between them.** Long-running chats accumulate huge histories that get resent on every turn. Start a fresh session for an unrelated task.
- **Commit often.** Frequent git checkpoints mean the agent rebuilds from a known-good state instead of redoing lost work.
- **Batch research.** Group lookups (as was done for resort/restaurant research) rather than many one-off round-trips.
- **Lean on free tooling first.** Linters, type-checkers, and tests catch issues deterministically — don't spend model tokens on what a compiler will tell you for free.

### 14.3 What to revisit post-launch

Once analytics (§13) show real usage: recompute cost-per-active-user against actual numbers, tune the §4.4 caps to real behaviour, and only then decide whether monetisation (§11.1) needs to move earlier.

---

## 15. Go-to-Market — First Customers & Early Growth

The launch strategy is a **deliberate, gradual warm-up** rather than a big public push. The goal in the early phase is **learning, not scale** — get the product in front of progressively wider (but still warm) audiences, watch how real people use it, and gather honest feedback before exposing it to cold, critical communities. This also keeps AI run-time costs (§4.4) naturally bounded while the product is still rough.

### 15.1 Expanding rings (widen one ring only when the previous one is healthy)

| Ring | Audience | How to reach them | What to learn |
|------|----------|-------------------|---------------|
| **0 — Friends** | Close friends who ski/snowboard or want to try Japan | Direct 1:1 share (message, in person) | Does the core flow make sense? What's confusing? Brutally honest UX feedback |
| **1 — Friends of friends** | Warm intros from Ring 0 | Ask Ring 0 to share with people planning a Japan trip | Does it hold up without you explaining it? First "stranger-ish" reactions |
| **2 — Facebook groups** | Ski/snowboard & Japan-travel groups (incl. Thai travellers to Japan) | Genuine, non-spammy posts — share as a helpful free tool, invite feedback | What real planners actually ask for; which resorts/questions dominate; demand signal |
| **3 — Reddit & wider social** | r/skiing, r/snowboarding, r/JapanTravel, etc. | Only once the product is polished — these communities are critical of self-promotion | Scale signal, SEO/word-of-mouth, edge-case feedback at volume |

> **Why gradual:** each ring is more critical and less forgiving than the last. Warm audiences give kinder, richer feedback and tolerate rough edges; cold communities (Reddit especially) punish unpolished or spammy launches. Earn the right to widen by fixing what each ring reveals first.

### 15.2 Readiness gate before widening a ring

Don't jump to the next ring on a schedule — jump when these are true:

- The **top issues** the current ring surfaced are fixed (or consciously deferred).
- Core flows work **without you in the room** to explain them.
- Analytics (§13) show people actually reaching resort detail pages and engaging — not just bouncing.
- For Rings 2–3: the product looks and feels **polished enough to share publicly** (design review + QA gates, §10.2).

### 15.3 The learning loop (this is the point of early launch)

For every ring, run a tight feedback cycle:

1. **Watch** — behavioural data via analytics (§13): where people drop off, which resorts/filters/questions dominate, what the AI chat gets asked (a goldmine for content gaps and the Phase 2 personalisation idea).
2. **Ask** — lightweight qualitative feedback: a short in-app prompt, a quick chat with Ring 0–1 users, a "what were you looking for that you didn't find?" question.
3. **Learn** — feed insights back into content (`content-model.md` gaps), features (roadmap §10), and copy/tone. Early users effectively co-design the product.
4. **Iterate, then widen** — close the loop before opening the next ring.

### 15.4 Tone when sharing

Consistent with the product's voice (§12 — "a well-travelled friend giving honest advice"): share the product as a **genuinely useful free tool**, not a sales pitch. Lead with helpfulness, invite criticism, and be visibly responsive to feedback. This builds the trust that turns early users into the word-of-mouth that powers Rings 2–3.

---

## 16. Risk Register

The most significant risks to the product, with direction for mitigation. Likelihood/Impact are rough (L/M/H) to help prioritise. Review this list at each release gate (§10.2).

| # | Risk | L | I | Mitigation / direction |
|---|------|---|---|------------------------|
| R1 | **Unsplash changes API pricing or access terms** (it's the sole image source, §11.2) | M | H | Don't hard-couple to Unsplash: abstract image fetching behind an **internal image service/interface** so the provider can be swapped. Store selected `photo_id` + attribution metadata in `images.*` so we aren't re-querying live. Keep a fallback path (other free providers, or curated/partner photography in a later phase, §11.2). Monitor Unsplash API terms; treat images as a replaceable dependency, not a foundation. |
| R2 | **Google Maps API costs run higher than expected** | M | M | The architecture already allows **Leaflet.js + OpenStreetMap as a low/zero-cost alternative** (§7.1). Default to OSM where it's good enough; reserve Google Maps for features that truly need it. Set **billing alerts + a usage quota/cap** in the Google Cloud console. Lazy-load maps (only init on interaction), cache Places results, and avoid per-page-load map calls. |
| R3 | **AI API costs become unsustainable in Phase 3 without revenue** | M | H | Enforce the §4.4 guardrails (turn/token caps, per-IP cap, **global daily kill-switch**) and §14 optimisation (seed-content-first, shared-cache snow, prompt caching, model routing). Worst case is **bounded by design** — it degrades to a non-AI fallback rather than overspending. Recompute cost-per-user against real analytics (§13) before any growth push; bring monetisation (§11.1) forward if the data demands it. |
| R4 | **Content production falls behind development, delaying launch** | M | M | Content is decoupled from product (separate files, §8) and **most of it (T1) can be produced off-season** (§4.3, `content-model.md`). Use the **content-freeze gate** (§10.2) as the real launch dependency. **All 20 resorts must reach `content_status: published` before Phase 1 launch** — use the production-order list (§8) to sequence research, but do not ship with fewer than 20. AI-assisted research keeps production fast. |
| R5 | **Resort data becomes inaccurate and damages trust** | M | H | Trust is the core value prop. Mitigations: the **volatility-tier model** keeps T3 data live (never stale) and T2 labelled by season (§4.3); the agent answers from verified seed content first (§4.1); every resort carries `data_confidence`, `last_reviewed`, and `sources` (§8.1); the UI surfaces a **season label + "last reviewed" signal** (§11.3); the seasonal refresh re-validates T2 facts (§8.1 / `content-model.md`). Honest "lowlights" and "watch out for" content reinforce credibility over hype. |

> **General posture:** every paid external dependency (Unsplash, Google Maps, Anthropic) is treated as **swappable and cost-capped**, never as an unbounded foundation. Set billing alerts on all three before public launch.

---

## 17. Legal & Compliance

These are **launch-blocking** requirements — they must be live before the product is shared publicly (Ring 2+ in §15), not deferred. GDPR/PDPA handling is referenced in §6 and §11.4; this section makes the concrete deliverables explicit.

### 17.1 Required pages (before public launch)

- **Privacy Policy page** — what data is collected (analytics events §13, account data if/when accounts exist §11.4), legal basis, third parties (Anthropic, Google, Unsplash, PostHog/Plausible, Supabase), cookie/consent details, data-retention period, and user rights (access/erasure) with a contact method. Must satisfy both **GDPR** and **Thailand PDPA**.
- **Terms of Service page** — usage terms, the **"information is provided as-is, verify against official resort sources" disclaimer** (directly supports R5 and §11.3), limitation of liability, acceptable use, and governing law.
- **Cookie / consent banner** — already required in §6; must gate analytics/identified tracking until consent is given (ties to §13.1 privacy-first default and `consent_state`).

### 17.2 Unsplash attribution (legally required — not optional)

The Unsplash API license **requires crediting the photographer on every image used**. This is a hard requirement of the §11.2 decision to use Unsplash. Full spec: [Unsplash attribution guideline](https://help.unsplash.com/en/articles/2511315-guideline-attribution).

- Display **photographer name + link to their Unsplash profile** and a **link to Unsplash** on or adjacent to every image — visible, not buried.
- **All links must include UTM params:** `?utm_source=japan_ski_planner&utm_medium=referral` (use the app name; see `.env.example` `UNSPLASH_UTM_SOURCE`).
- **Recommended copy:** *Photo by [Name] on Unsplash*
- **HTML template (implement as a reusable component):**
  ```html
  Photo by <a href="{photographer_url}" rel="noopener noreferrer">{photographer}</a> on <a href="https://unsplash.com/?utm_source=japan_ski_planner&utm_medium=referral" rel="noopener noreferrer">Unsplash</a>
  ```
- **Store full attribution metadata in `images.*`** (`photographer`, `photographer_url`, `photo_url`, `photo_id`) — see `content/site-images.md` (home hero example) and per-resort `content/resorts/<slug>.md`.
- Trigger Unsplash's **download endpoint** (`GET /photos/:id/download`) when an image is first used in the app — tracking only, not for embedding (use CDN/`urls.regular` for display).
- This applies everywhere images appear: hero images, village/food photos, cards, carousels.

> **Early dev:** the home hero is pre-selected in `content/site-images.md` (Karsten Winegeart / `L7289nHzVgI`) so mockups can ship before Unsplash app approval. Bulk resort fetch via `scripts/fetch_unsplash_assets.py` stays on hold until a **site screenshot** is available for the developer application.

### 17.3 Data handling principles

- **Data minimisation:** collect only what's needed (§6). No PII in analytics event properties (§13.1).
- **Consent-gated tracking:** anonymous/cookieless by default; identified tracking only post-consent.
- **Third-party data flows documented:** maintain the list of processors (Anthropic, Google Places, Unsplash, analytics, auth/DB) in the Privacy Policy and keep it current as the stack changes.
- **Right to erasure:** once accounts exist (§11.4), provide an account-deletion path that removes user data and aliased analytics identity.

> **Launch checklist tie-in:** Privacy Policy, Terms of Service, cookie consent, and Unsplash attribution are part of the **launch gate** (§10.2) — public sharing in §15 cannot begin until all four are live.

---

## 18. Glossary

| Term | Definition |
|------|------------|
| **T1 / T2 / T3** | Content volatility tiers (`content-model.md` §2). T1 = evergreen; T2 = seasonal (label the season); T3 = live only, never stored. |
| **`resort_slug`** | Kebab-case file identifier, e.g. `niseko-united`, matching `content/resorts/<slug>.md`. |
| **Gateway city / airport** | Arrival hub for a resort (e.g. CTS → Sapporo for Hokkaido resorts). |
| **Valley pass** | Multi-resort lift pass covering interconnected areas (e.g. Hakuba Valley). |
| **Trail map / piste map** | Official resort slope map; JP colour convention: green = beginner, red = intermediate, black = advanced. |
| **Seed content** | Structured + narrative resort data in markdown frontmatter/body (§8.1). |
| **Plan tab** | Bottom-nav workspace for saved trips — **Phase 3** only (§11.4). |
| **Ring 0–3** | GTM audience expansion stages (§15): friends → friends-of-friends → Facebook → Reddit/social. |
| **FR-n** | Phase 1 functional requirement ID (§3). |
| **Guardrail metric** | Counter-metric in §2 that must not regress while optimising north-star metrics. |

---

*Document prepared for product planning purposes. All resort data listed is approximate and must be verified against official resort sources before publication.*
