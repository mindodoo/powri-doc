# Epic 7: Phase 1 UX Wave 2 (Post-Launch Polish)

**Status:** Backlog · **Phase:** 1.2 (post-launch) · **Runs after:** Epic 6 complete (`main` in production)  
**Goal:** Fix discovery/navigation friction, quiz flow polish, responsive spacing, terrain chip readability, and “Around resort” content accuracy.

> **AI agents:** Read this shard only (not full `epics.md`). One story per session. Update `sprint-status.yaml` when starting/completing a story.

---

## Overview

| Type | Stories | Description |
|------|---------|-------------|
| **Feature / UX** | 7.1, 7.3, 7.4 | Quiz slide transitions, overlay back button, top bar + hamburger + inline search |
| **Cosmetic / layout** | 7.2, 7.1 (results spacing) | Global top/side spacing; quiz results header on small screens |
| **Content + feature** | 7.5, 7.6 | Shorter terrain stat chips; “Around resort” area label + daytrip editorial |

**Out of this epic:** AI chat, accounts, maps, user reviews (Phase 3).

---

## Requirements map (product owner)

| ID | Source | Story |
|----|--------|-------|
| PO-7 | Quiz: bottom nav feels like it refreshes on each answer; want in-page slide transition | **7.1** |
| PO-7b | Quiz results: too much top margin; #1 card clipped on iPhone SE | **7.1** |
| PO-8 | Many pages: excessive top margin; side margins wrong on iPad/desktop | **7.2** |
| PO-9 | Back button should always overlay top-left | **7.3** |
| PO-10 | Remove app title + “Find your resort” from top bar; hamburger menu; inline search | **7.4** |
| PO-11 | Terrain/mountain stat chips too long → horizontal scroll | **7.5** |
| PO-12 | “Around {city}” shows airport (e.g. Narita) not resort area (e.g. Yamagata) | **7.6** |

---

## Current state (for implementers)

### Quiz question transitions (PO-7)

`QuizFlow` advances `questionIndex` in React state — **no route change**. Each answer re-renders `ResortQuizCard` with a **new `key`** (`${mode}-${questionIndex}`), which **unmounts/remounts** the card and replays the `quiz-slide-in` CSS animation on the **entire** question block (`ResortQuizCard.tsx`, `globals.css`).

`BottomNav` lives in `AppShell` and should not remount; the perceived “refresh” is likely **layout shift** from remounting the question card + `min-h-[calc(100dvh-7rem-…)]` on the quiz column. Fix: **single persistent quiz stage** with slide-out/slide-in between questions (no full-card remount).

### Quiz results top spacing (PO-7b)

Results stack: `QuizBackButton` (in-flow padding) + `QuizResultsReveal` with `py-section-gap` (48px) + header before `#1` card. On **375×667 (iPhone SE)**, the podium card can be **partially clipped** above the fold.

### “Around resort” bug (PO-12)

`GatewayExperiencesBlock` renders `t('gatewayCity', { city: gatewayAirport })` — i.e. **airport name**, not resort area. Example: Zao Onsen (`gateway_airport: Sendai`) shows **“Around Sendai”** while daytrips are Yamagata-area. Airport context already appears in `GettingThereSection` via `gatewayFrom`.

### Top bar (PO-10)

- Home: `HomeTopBar` → `TopBarWithSearch` with `metadata.title` (“Japan Ski Trip Planner”) + search icon toggle.
- Explore: `ExploreTopBar` with `explore.title` (“Find your resort”) + search icon toggle.
- Search opens `HomeSearchPanel` dropdown — not an always-visible field.

### Spacing (PO-8)

- `--space-section-gap: 3rem` (48px) used widely (`py-section-gap`).
- Content columns use `max-w-lg` (512px) centered — on tablet/desktop this reads as **excessive side gutters**.
- Explore adds `pt-[calc(3.5rem+safe-area)]` **plus** `pt-section-gap` on inner header.

---

## Recommended implementation order

```
7.2 (tokens — unblocks 7.1 results spacing) ──┐
7.3 (overlay back — shared by quiz/detail) ──┼──► 7.1 (quiz transitions + results layout)
7.4 (top bar — larger; can parallel after 7.2) ──┘
7.5, 7.6 (content — can parallel; 7.6 needs schema + i18n)
```

**Suggested first story for quick win:** **7.6** (clear user-visible bug) or **7.2** (foundation for layout stories).

---

## Story 7.1: Quiz In-Page Slide Transitions & Results Spacing {#story-71}

**Type:** `[feature]` + `[cosmetic]` · **FR:** IMP-13 · **Depends on:** 7.2 (spacing tokens), 7.3 (overlay back — recommended)

As a **quiz player**,  
I want questions to slide horizontally within one screen without the app chrome jumping,  
So that the quiz feels smooth and the results show my #1 match fully on small phones.

### Acceptance criteria

**Given** I am on `/quiz` in the **questions** phase  
**When** I tap an answer option  
**Then** the **current question slides out to the left** and the **next question fades/slides in from the right** within a **single persistent container** (no remount via changing `key` on the whole card)

**And** `BottomNav` does **not** visually flash, jump, or re-animate on answer tap (verify on iPhone SE viewport)

**And** progress text (`Question X of 4`) updates without layout shift of the bottom nav

**And** `prefers-reduced-motion: reduce` disables slide animation (instant swap or cross-fade only)

**And** existing quiz analytics unchanged (`quiz_started`, `quiz_completed`)

**Given** quiz **results** on a **375×667** viewport  
**When** results render (Serious mode, after reveal)  
**Then** the **#1 match card** is fully visible without the top half clipped — adjust top padding/margins (coordinate with **7.2** compact section gap and **7.3** overlay back)

**And** results title (`quiz.results.serious` / cosmic) remains readable with **≤24px** top gap below the overlay back control (not 48px+ stacked padding)

### Implementation hints

| Area | Path |
|------|------|
| Quiz orchestration | `web/src/components/quiz/QuizFlow.tsx` |
| Question UI | `web/src/components/quiz/ResortQuizCard.tsx` |
| Animations | `web/src/app/globals.css` — add `quiz-slide-out-left`, `quiz-slide-in-right`; respect `prefers-reduced-motion` |
| Pattern | Hold `displayQuestionIndex` + `animatingToIndex` state; or CSS transform on a horizontal track |
| Remove | `key={\`${mode}-${questionIndex}\`}` remount pattern on full card |

### Validate

- Manual: 4 taps through quiz on iPhone SE — no bottom nav flicker; results #1 fully visible
- `npm run test:quiz` (if present) + `npm run lint` + `npm run build`

---

## Story 7.2: Global Page Spacing — Top & Responsive Horizontal {#story-72}

**Type:** `[cosmetic]` · **FR:** IMP-14 · **Depends on:** none

As a **visitor on any device**,  
I want page content to sit closer to the top and use screen width sensibly on tablet/desktop,  
So that I see more content without odd empty margins.

### Acceptance criteria

**Given** design tokens in `tokens.css`  
**When** updated  
**Then** `--space-section-gap` is reduced (target **2rem / 32px**, or add `--space-section-gap-compact: 1.5rem` for inner page headers)

**And** a responsive **content max-width** token exists, e.g. `--content-max-width: 32rem` (512px) on mobile, **`42rem` (672px) at `md`**, **`48rem` (768px) at `lg`** — applied via shared utility or `PageContent` wrapper

**And** horizontal padding `--space-screen-x` scales: **20px** mobile, **24px** `md+` (not wider than 24px on desktop — avoid “double gutter” with max-width)

**Given** these pages at **768px** and **1280px** width  
**When** rendered  
**Then** top spacing is **visibly less** than current production on: Home (discovery block), Explore, Info, Quiz (intro + results), Legal, Resort detail content below hero

**And** Explore page does **not** stack `pt-[calc(3.5rem+safe-area)]` **and** `pt-section-gap` on the same header block (pick one clear offset below top bar)

**And** no overlap regressions with fixed top bar (**7.4**) or overlay back (**7.3**) — document offset in shared constant e.g. `lib/layout/pageInsets.ts`

### Implementation hints

| Area | Path |
|------|------|
| Tokens | `web/src/styles/tokens.css`, `globals.css` `@theme` |
| Pages to audit | `page.tsx` (home), `explore/page.tsx`, `info/page.tsx`, `quiz/*`, `ResortDetailContent.tsx`, `LegalPageLayout.tsx` |
| Wrapper (optional) | `components/layout/PageContent.tsx` — `mx-auto w-full max-w-* px-screen-x` |

### Validate

- Visual check: iPhone SE, iPad portrait, desktop 1280px
- `npm run build`

---

## Story 7.3: Overlay Back Button — All Pages {#story-73}

**Type:** `[feature]` + `[cosmetic]` · **FR:** IMP-15 · **Depends on:** none (pairs with 7.1, 7.2)

As a **user on any screen**,  
I want the back control fixed at the top-left over the page content,  
So that I can exit consistently without scrolling or losing it to layout shifts.

### Acceptance criteria

**Given** pages that show a back control today  
**When** rendered  
**Then** back is **`position: fixed`** (or `sticky` with equivalent visual) at **top-left**, respecting `env(safe-area-inset-top)` + `px-screen-x`

**And** `z-index` is above page content but below modals/consent banner (document layer: back ≈ 45, top bar 40, bottom nav 50)

**And** a **single shared component** replaces ad-hoc patterns: `QuizBackButton`, in-flow detail back, etc.

| Surface | Back behaviour |
|---------|----------------|
| Quiz (intro, questions, results) | → `/explore` (keep **6.4** behaviour) |
| Resort detail | `router.back()` or `/` fallback (keep **ResortDetailBack** logic) |
| Legal (Privacy, Terms) | → `/info` or back |
| Onboarding | unchanged skip flow (no back required unless PO adds) |

**And** page content below the back control has **top padding** so titles are not hidden under the button (coordinate padding with **7.2**)

**And** detail hero: back remains **on the image** (white on scrim) — same component, `variant="onImage"` prop

**And** min tap target **44×44px**; `aria-label` from i18n

### Implementation hints

- New: `web/src/components/layout/OverlayBackButton.tsx` (or extend `QuizBackButton` + `ResortDetailBack`)
- Quiz: remove in-flow back wrapper; use fixed overlay
- Detail: `ResortDetailHero.tsx` — swap inline button for shared component

### Validate

- Manual: quiz all phases, resort detail, legal pages — back always top-left, no overlap with h1

---

## Story 7.4: Top Bar — Hamburger Menu & Inline Search {#story-74}

**Type:** `[feature]` · **FR:** IMP-16 · **Depends on:** 7.2 (top offsets)

As a **planner**,  
I want a compact top bar with navigation menu and an always-visible search field,  
So that I can find resorts without redundant titles in the header.

### Acceptance criteria

**Given** Home and Explore (and any page using the discovery top bar)  
**When** the top bar renders  
**Then** the text **“Japan Ski Trip Planner”** does **not** appear in the header

**And** the text **“Find your resort”** does **not** appear in the Explore **top bar**

**And** **“Find your resort”** appears as the **page hero `h1`** on Explore (move from `ExploreTopBar` title into page body; keep eyebrow/subtitle)

**Given** the top bar layout  
**When** rendered at 375px width  
**Then** layout is: **[Hamburger]** · **[Search field ……………………]** spanning to the right edge

**And** hamburger opens a **menu** (sheet or dropdown) with at least: **Home**, **Explore** (resorts), **Info** — labels from `nav.*` i18n

**And** search field shows placeholder **“What are you looking for”** (`search.placeholderDiscover` or update `search.placeholder`) in **grey** (`text-text-secondary`)

**And** search behaviour reuses existing resort name search (`HomeSearchPanel` logic / `ResortSearchItem`) — typing filters or submits to existing search UX (inline results below field or expandable panel — match current search API)

**Given** Home with hero carousel  
**When** at top of page  
**Then** top bar can remain **transparent over hero** (search field: subtle border/bg for legibility on photo); solid `bg-surface` when scrolled (preserve current `HomeTopBar` scroll behaviour where possible)

**And** Explore top bar is **always solid** (no hero)

**And** Quiz, resort detail, Info, Legal: **no duplicate** hamburger/search unless PO adds later — scope is Home + Explore only for this story

**Optional analytics:** `nav_menu_opened`, `search_focused` — add to `docs/tracking-plan.md` if implemented

### Implementation hints

| Area | Path |
|------|------|
| Refactor | `TopBarWithSearch.tsx`, `HomeTopBar.tsx`, `ExploreTopBar.tsx` |
| New | `AppMenuSheet.tsx` or `NavDrawer.tsx` |
| Search | Extract shared logic from `HomeSearchPanel.tsx` → `ResortSearchField.tsx` |
| i18n | `web/messages/en.json` — `search.placeholder`: "What are you looking for" |
| Explore page | `explore/page.tsx` — add `<h1>{t('title')}</h1>` in body |

### Validate

- Manual: Home + Explore at 375 / 768 / 1280; menu navigates; search finds resort by name
- `npm run lint` + `npm run build`

---

## Story 7.5: Terrain & Mountain Stat Chips — Shorter Tags {#story-75}

**Type:** `[content]` + `[cosmetic]` · **FR:** IMP-17 · **Depends on:** none

As a **detail page reader**,  
I want terrain and mountain data chips to be short scannable tags,  
So that I do not need horizontal scrolling to read stats on mobile.

### Acceptance criteria

**Given** `StatRow` chips on resort detail (season snapshot + terrain stats rows)  
**When** rendered at **375px** width  
**Then** **no horizontal scroll** is required for the stat chip row on any published resort

**And** each chip label/value pair targets **≤28 characters** visible text (or ≤2 short words for `snow_quality` / `snowfall` chips)

**And** long prose fields are **not** shown as chips — move to body text below chips if needed:
- `snow_quality`, `terrain_parks`, `night_skiing`, `tree_runs_offpiste_policy` when >28 chars

**Content editorial (all 20 resorts):** shorten frontmatter values used in chips, e.g.:
- ❌ `Heavy Tohoku snow up high; can be icy/wind-affected; rime-ice 'snow monsters'`
- ✅ `Heavy Tohoku snow` or `Rime-ice monsters`

- ❌ `8 patrol-managed freeride zones covering most of the mountain…`
- ✅ `8 gated freeride zones`

**Optional UI guardrail:** `StatRow` applies `max-w-[…] truncate` with `title={fullValue}` for overflow — but **content shortening is required** for AC

### Implementation hints

| Area | Path |
|------|------|
| Chips UI | `web/src/components/resort/StatRow.tsx` |
| Data mapping | `web/src/lib/resort/detailSections.ts` — split long values to `terrainFacts` dd list instead of chips |
| Content | `content/resorts/*.md` — `snow_quality`, season stats, `terrain_parks`, etc. |
| QA | Grep resorts for chip sources >40 chars |

### Validate

- Manual: worst offenders (Lotte Arai, Zao, Kandatsu) at 375px — no chip row scroll
- `npm run build`

---

## Story 7.6: “Around Resort” Section — Area Label & Daytrip Content {#story-76}

**Type:** `[content]` + `[feature]` · **FR:** IMP-18 · **Depends on:** none

As a **detail page reader**,  
I want the daytrip section titled for the **resort area**, not the gateway airport,  
So that “Around Zao” means Yamagata/Zao Onsen — not Sendai or Narita.

### Acceptance criteria

**Given** resort frontmatter  
**When** the getting-there **daytrips block** renders  
**Then** the section heading uses **`around_area`** (new field) or derived label — **not** `gateway_airport`

**Priority for area label:**
1. `around_area` (new optional string, editorial)
2. `nearest_village` if set
3. `nearest_town`
4. `prefecture` + "area" (e.g. "Yamagata area")
5. Omit section heading city if none apply (list cards only)

**And** i18n key updated: `resortDetail.aroundArea` → **"Around {area}"** (deprecate `gatewayCity` for this block)

**And** `gateway_airport` remains only in the **gateway line** at top of Getting There (`gatewayFrom` — "From Narita / Haneda…")

**And** `nearby_daytrips` entries describe **resort-area** experiences (towns, onsen, linked resorts, food) — **not** airports or "via Tokyo" transport hubs

**Editorial pass (all 20 resorts):**
- Zao Onsen: heading **"Around Zao Onsen"** or **"Around Yamagata"** — not Sendai
- Joetsu Kokusai / Naeba: **Yuzawa / Minami-Uonuma area** — not Narita
- Remote resorts with sparse surroundings: wider prefecture or adjacent prefecture is OK (e.g. "Around Hachimantai" for Appi)

**And** `experienceFallback` copy no longer references gateway airport

### Data model

Add to `web/src/lib/content/schema.ts` + `CONTENT_MODEL.md` §4:

```yaml
around_area: "Zao Onsen"  # optional [T1] — local area name for daytrips heading
```

Populate for all resorts (can match `nearest_village` or `nearest_town` where obvious).

### Implementation hints

| Area | Path |
|------|------|
| Block | `web/src/components/resort/GatewayExperiencesBlock.tsx` — rename prop `areaLabel` |
| Section | `GettingThereSection.tsx` — pass `aroundArea` not `gatewayAirport` |
| Loader | `loadResorts.ts`, `detailSections.ts` |
| i18n | `web/messages/en.json` — `aroundArea` |
| Content | `content/resorts/*.md` |

### Validate

- Spot-check: Zao, Joetsu Kokusai, Niseko, Appi — heading matches local geography
- `npm run test:launch` + `npm run build`

---

## Epic 7 exit criteria

- [ ] Stories 7.1–7.4 merged (UX/navigation)
- [ ] Stories 7.5–7.6 merged (content + chips + around area)
- [ ] `docs/launch-checklist.md` §4 spot-check on iPhone SE + iPad
- [ ] Sprint status: `epic-7: done`
- [ ] Epic retro: `epic-7-retro-YYYY-MM-DD.md`

---

**PRD shard:** [`../prd/17-phase1-improvements-wave2.md`](../prd/17-phase1-improvements-wave2.md)  
**Prior epic:** [`epic-06-phase1-improvements.md`](epic-06-phase1-improvements.md) (done)  
**Sprint:** [`../../implementation-artifacts/sprint-status.yaml`](../../implementation-artifacts/sprint-status.yaml)
