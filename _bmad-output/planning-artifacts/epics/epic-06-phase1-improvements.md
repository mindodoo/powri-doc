# Epic 6: Phase 1 Improvements (Post-Launch Polish)

**Status:** Backlog · **Phase:** 1.1 (pre–launch QA gate) · **Runs after:** Epic 5 complete  
**Goal:** UX polish, discovery flow refinements, content quality, and ops hygiene before manual launch QA.

> **AI agents:** Read this shard + linked feature briefs. One story per session (CS → DS → CR). Do **not** load full `epics.md`.

---

## Overview

| Type | Stories | Description |
|------|---------|-------------|
| **Feature change** | 6.1, 6.2, 6.4, 6.5 | Home list behaviour, quiz CTA, quiz nav, trail map copy |
| **Cosmetic / UX** | 6.3, 6.6, 6.12 | Quiz podium #1, highlight/lowlight icons, detail hero verdict size |
| **Content / editorial** | 6.7, 6.8, 6.9 | Unsplash images, daytrips, broken links |
| **Ops / analytics** | 6.10, 6.11 | PostHog dashboards, env/deploy docs |

**Out of this epic:** User-submitted resort reviews → [`phase-3-user-community.md`](phase-3-user-community.md) (requires accounts).

---

## Requirements map (product owner)

| ID | Source | Story |
|----|--------|-------|
| PO-1 | Home sort by popularity; 5 default; Show more → Explore | **6.1** |
| PO-2 | Quiz podium #1 — rank overlay on image, no side column | **6.3** |
| PO-3 | Home quiz CTA under hero | **6.2** |
| PO-4 | Quiz back arrow → Explore | **6.4** |
| PO-5 | Trail map — button only, no link in body text | **6.5** |
| RETRO-2 | Highlight/lowlight 16px icons (DESIGN.md) | **6.6** |
| RETRO-4 | Flagship resort Unsplash images | **6.7** |
| RETRO-5 | Daytrips for GALA + Joetsu Kokusai | **6.8** |
| RETRO-6 | Fix/verify link-probe URLs | **6.9** |
| RETRO-7 | PostHog baseline dashboards | **6.10** |
| RETRO-8 | Contact email env documented for deploy | **6.11** |
| PO-6 | Detail hero quick verdict — smaller type; no overlap with back button | **6.12** |

---

## Recommended implementation order

```
6.11 (tiny) ──┐
6.1 (data + home) ──► 6.2 (home CTA — same page)
6.3, 6.4 (quiz — can parallel)
6.5, 6.6 (detail polish — can parallel)
6.8, 6.9 (content — can parallel)
6.7 (images — editorial heavy; may span multiple sessions)
6.10 (PostHog — after deploy or with prod key)
```

**Before launch QA:** Complete at minimum **6.1–6.6** (product UX) + **6.11**. Content stories **6.7–6.9** strongly recommended. **6.10** can run on production project.

---

## Story 6.1: Home Resort List — Popularity Sort & Explore Handoff {#story-61}

**Type:** `[feature]` · **FR:** IMP-1 · **Depends on:** none

As a **first-time visitor**,  
I want the Home page to show the most popular resorts first and a short list,  
So that I see the best-known destinations immediately and browse the full catalog on Explore.

### Acceptance criteria

**Given** the Home discovery section  
**When** the page loads with filter "All"  
**Then** resorts are ordered by **`popularity_rank` ascending** (1 = most popular), **not** alphabetically by slug or name

**And** exactly **5** resort cards render by default on Home (not 10)

**And** a **"Show all resorts →"** (or equivalent) control appears when total published resorts > 5

**When** I tap that control  
**Then** I navigate to **`/explore`** (Explore tab) — the list does **not** expand in place on Home

**And** Explore still shows **all** resorts with existing filter/search behaviour unchanged

**And** analytics fires **`home_explore_cta_clicked`** with `{ source: 'home_show_all', visible_count: 5, total_count: 20 }` (add to `docs/tracking-plan.md` §2)

**And** Home no longer fires `load_more_clicked` for in-page expansion (Explore retains load-more if applicable)

### Data model

Add **`popularity_rank`** (integer 1–20, unique per published resort) to resort frontmatter schema (`web/src/lib/content/schema.ts`, `CONTENT_MODEL.md` §4).

- **1** = highest international recognition (e.g. Niseko United)
- Editorial default order documented in story artifact when populated
- `getAllResorts()` or `toResortListItems()` applies sort: `popularity_rank` ASC, then `nameEn` as tiebreaker
- Resorts missing rank: warn at build; fall back to end of list

### Implementation hints

| Area | Path |
|------|------|
| Sort | `loadResorts.ts` or new `sortResortsByPopularity.ts` |
| Home list cap | `ResortList` — `initialVisible={5}` on Home only; Explore unchanged |
| Show all CTA | Replace `handleShowMore` on Home with `<Link href="/explore">` |
| i18n | `home.showAllResorts` in `web/messages/en.json` |

### Validate

- `npm run build` + manual: Home shows 5, order matches rank, CTA → Explore tab active
- Update `verify-launch-gates.ts` if popularity_rank required for published resorts

---

## Story 6.2: Home Quiz CTA Under Hero {#story-62}

**Type:** `[feature]` · **FR:** IMP-2 · **Depends on:** none (pairs well after 6.1)

As an **undecided planner**,  
I want a clear quiz entry point on Home below the hero,  
So that I can start "Find my resort" without opening Explore first.

### Acceptance criteria

**Given** the Home page below `HeroCarousel` and above the resort discovery heading  
**When** the section renders  
**Then** it shows heading copy equivalent to **"Find your right ski resort"** (i18n key `home.quizCtaHeading`)

**And** a primary button **"Find my resort"** (reuse existing quiz CTA copy where possible) links to **`/quiz`**

**And** tapping fires **`quiz_started`** is **not** fired here — only on quiz intro start (existing behaviour)

**And** optional analytics: **`quiz_cta_tapped`** with `{ entry_point: 'home_hero', surface: 'home' }` (add to tracking plan §2)

**And** layout respects mobile 375px: full-width button, min tap target 52px, spacing per Theme A tokens

**And** section does not duplicate Explore tab quiz CTA (Home-specific placement only)

### Implementation hints

- New `HomeQuizCta.tsx` inserted in `web/src/app/[locale]/page.tsx` between hero and discovery block
- Match accent button style from Explore `FindMyResortCta` if one exists

---

## Story 6.3: Quiz Podium #1 — Rank Overlay on Image {#story-63}

**Type:** `[cosmetic]` · **FR:** IMP-3 · **Depends on:** none · **Brief:** [`../features/quiz.md`](../features/quiz.md)

As a **quiz completer**,  
I want the #1 result to look like a clear winner without a side label column,  
So that the card feels visual-first like other resort cards.

### Acceptance criteria

**Given** quiz results with `QuizTopMatchCard` for rank #1  
**When** results render (Serious or Cosmic, after reveal)  
**Then** the **left side column** showing rank **"1"** and **"Your top match"** / cosmic variant text is **removed**

**And** a **"1"** badge overlays the **top-left corner** of the resort **image** (inside image bounds, above gradient if any)

**And** badge styling: readable on photo and placeholder (white/semi-opaque pill or accent circle per DESIGN.md; min 32×32px tap-safe if interactive — badge is decorative only)

**And** card retains: image, region, name, match reason, tagline, badges, chevron — full-width single column layout

**And** `resort_card_tapped` with `position: 1`, `list_context: quiz` unchanged

**And** remove or repurpose i18n keys `quiz.results.topMatchLabel` / `topMatchLabelCosmic` if unused

### Current state

`QuizTopMatchCard.tsx` uses a fixed `w-12` left rail with "1" + label text — replace with image overlay.

---

## Story 6.4: Quiz Back Navigation to Explore {#story-64}

**Type:** `[feature]` · **FR:** IMP-4 · **Depends on:** none

As a **user deep in the quiz**,  
I want a back control that returns me to Explore,  
So that I can exit the quiz flow without using browser back or bottom nav alone.

### Acceptance criteria

**Given** I am on `/quiz` in **intro**, **questions**, or **results** phase  
**When** the screen renders  
**Then** a **back arrow** (consistent with `ResortDetailBack` pattern) appears top-left

**When** I tap back  
**Then** I navigate to **`/explore`** (not Home)

**And** back control respects safe-area inset top

**And** on **results** phase, back goes to Explore (does not restart quiz — acceptable; session state may reset on remount)

**And** accessibility: `aria-label` from i18n (`quiz.backToExplore`)

**Optional:** track **`quiz_abandoned`** with `{ phase, question_index }` — defer unless PO requests; not required for AC

### Implementation hints

- `QuizBackButton.tsx` in quiz components; mount in `QuizFlow` for all phases
- Use `Link` from `@/i18n/navigation` or project locale-aware router

---

## Story 6.5: Trail Map Section — Button Only, No Inline Links in Copy {#story-65}

**Type:** `[feature]` + `[content]` · **FR:** IMP-5 · **Depends on:** none

As a **detail page reader**,  
I want the trail map section to use the CTA button only,  
So that I am not confused by duplicate link mentions in the descriptive text.

### Acceptance criteria

**Given** a resort detail trail map section  
**When** rendered  
**Then** the **"View trail map"** button remains the **only** clickable outbound link in that section

**And** the trail map section shows **only** the "View official trail map" button — no descriptive note paragraph in the UI (copy may remain in `trail_map_note` for editorial reference)

**And** `trail_map_note` prose in content **does not contain raw URLs** or domain names

**And** if note text previously contained a URL, content files are updated to plain language (all 20 resorts audited)

**And** `trail_map_opened` + `external_link_clicked` still fire **only** on button click

**And** section remains accessible: note as paragraph, button with `Map` icon unchanged

### Content task

Batch-edit `content/resorts/*.md` `trail_map_note` fields — remove embedded domain names used as de facto links where they read like CTAs; keep factual map format descriptions.

---

## Story 6.6: Highlight / Lowlight Row — Column Header Icons {#story-66}

**Type:** `[cosmetic]` · **UX:** UX-DR8 polish · **Depends on:** none

As a **detail page reader**,  
I want visual cues on the highlights and watch-outs column headers,  
So that the honesty grid scans faster on mobile without cluttering each bullet.

### Acceptance criteria

**Given** `HighlightLowlightRow` with non-empty highlights or lowlights  
**When** rendered  
**Then** each column header ("Highlights" / "Watch out for") includes a **16px Lucide icon** before the label text

**And** highlights header uses a positive/neutral icon (e.g. `CircleCheck` or `Sparkles`) in `{colors.accent}`

**And** lowlights header uses a caution icon (e.g. `AlertTriangle`) in `{colors.accent-warm}`

**And** bullet rows remain **text-only** (no per-item icons)

**And** no layout regression at 375px (50/50 columns preserved)

### What this is (PM note)

Epic 3 shipped **text-only** column headers and bullet rows. This story adds a single icon per column header for scanability — not on every list item, which reads as redundant on mobile. **Purely cosmetic**; no content schema change.

---

## Story 6.7: Flagship Resort Unsplash Images {#story-67}

**Type:** `[content]` + `[feature]` · **NFR-7** · **Depends on:** Unsplash app approval (see `content/site-images.md`)

As a **visitor**,  
I want real resort photography on flagship destinations,  
So that discovery and detail feel credible before all 20 are illustrated.

### Acceptance criteria

**Phase A — minimum (launch polish):**  
**Given** Niseko United, Hakuba Valley, Nozawa Onsen (minimum)  
**When** content is updated  
**Then** each has ≥1 **`images.hero`** entry with full Unsplash attribution metadata (`photo_id`, `photographer`, `photographer_url`, `photo_url`, `alt`)

**And** `images.card` populated (or hero[0] fallback documented)

**And** detail hero + list cards show real images with **`UnsplashAttribution`** visible

**Phase B — stretch:** remaining 17 resorts (separate sub-task or follow-up story 6.7b if needed)

### Validate

- `npm run test:launch` — reduce "no Unsplash images" warnings for completed slugs
- Trigger Unsplash download endpoint on first use per §17.2 when API key available

---

## Story 6.8: Gateway Daytrips — GALA Yuzawa & Joetsu Kokusai {#story-68}

**Type:** `[content]` · **FR-3 polish** · **Depends on:** none

As a **reader on thinner Getting There pages**,  
I want gateway experience cards where other resorts have them,  
So that the detail page feels consistent.

### Acceptance criteria

**Given** `gala-yuzawa.md` and `joetsu-kokusai.md`  
**When** editorial pass completes  
**Then** each has **≥3** entries in **`nearby_daytrips`** (same format as other resorts)

**And** `GatewayExperiencesBlock` renders cards on both detail pages (currently hidden when <3)

**And** `last_reviewed` may be stamped as part of this edit (optional)

---

## Story 6.9: External Link Verification & Fixes {#story-69}

**Type:** `[content]` · **Depends on:** none

As a **product owner**,  
I want trail map and source URLs verified after automated probe failures,  
So that users do not hit dead links at launch.

### Acceptance criteria

**Given** `npm run test:launch:links` reported failures (2026-06-14):  
- Appi Kogen `trail_map_url` / sources  
- Joetsu Kokusai `trail_map_url`  
- Kandatsu Kogen `trail_map_url` + source  
- Lotte Arai Myoko tourism source  

**When** each URL is checked in a **real browser**  
**Then** replace with working official URL **or** document 403-as-bot-block if URL works in browser (update launch checklist note)

**And** re-run `test:launch:links` — zero new errors for replaced URLs

---

## Story 6.10: PostHog Baseline Dashboards {#story-610}

**Type:** `[ops]` · **NFR-13** · **Depends on:** PostHog live (done)

As a **product owner**,  
I want baseline dashboards in PostHog,  
So that launch metrics are observable from day one.

### Acceptance criteria

**Given** PostHog project (EU) with live Phase 1 events  
**When** dashboards are configured per `docs/tracking-plan.md` §5  
**Then** these views exist (create or confirm wizard-created):

1. **Discovery funnel** — `$pageview` (home) → `resort_card_tapped` → `resort_detail_viewed`
2. **Filter guardrail** — `filter_zero_results` trend
3. **External link CTR** — `external_link_clicked` by `link_type`
4. **Quiz funnel** — `quiz_started` → `quiz_completed`
5. **Satisfaction** — `satisfaction_prompt_submitted` avg score

**And** dashboard URLs recorded in `docs/analytics-verification.md` or `posthog-setup-report.md`

**Note:** Primarily **human configuration** in PostHog UI; no app code unless insight-as-code export desired.

---

## Story 6.11: Deploy Env — Contact Email & Env Documentation {#story-611}

**Type:** `[ops]` · **Depends on:** none

As a **deployer**,  
I want contact email and analytics env vars documented in one place,  
So that production Info tab and PostHog work on first deploy.

### Acceptance criteria

**Given** `web/.env.example` already lists `NEXT_PUBLIC_CONTACT_EMAIL`  
**When** docs are synced  
**Then** [`docs/launch-checklist.md`](../../../docs/launch-checklist.md) §deploy lists all required production env vars with descriptions

**And** root [`.env.example`](../../../.env.example) cross-links to `web/.env.example` for app vars (or duplicates `NEXT_PUBLIC_CONTACT_EMAIL` with comment)

**And** Info page shows real mailto when `NEXT_PUBLIC_CONTACT_EMAIL` set (existing behaviour verified)

---

## Story 6.12: Detail Hero Quick Verdict — Smaller Type {#story-612}

**Type:** `[cosmetic]` · **UX:** UX-DR7 polish · **Depends on:** none

As a **detail page reader**,  
I want the quick verdict pull quote in a smaller type size,  
So that long resort summaries do not push the resort name into the back-button area on mobile.

### Acceptance criteria

**Given** a resort detail hero with a quick verdict (markdown `## Quick verdict`)  
**When** rendered at 375px width  
**Then** verdict text uses the **`quick-verdict` token** (13px — smaller than former 24px `display-italic`)

**And** verdict is clamped to **3 lines** max so the hero title stack stays clear of the top back control

**And** resort name (`h1`) and back button remain visually separated (no overlap)

**And** italic serif styling and white-on-gradient legibility preserved

### Content task (same story)

- **Joetsu Kokusai:** `name_en` → `Joetsu Kokusai` only; remove PRD/Nendai naming notes from frontmatter and body copy

---

## Epic 6 exit criteria

- [x] Stories 6.1–6.6 merged and verified on `main`
- [x] Content stories 6.7–6.9 complete or explicitly deferred with PO sign-off
- [ ] `docs/launch-checklist.md` §4 manual QA executed
- [x] Sprint status: `epic-6: done`

---

**PRD shard:** [`../prd/16-phase1-improvements.md`](../prd/16-phase1-improvements.md)  
**Phase 3 reviews (PO-5):** [`phase-3-user-community.md`](phase-3-user-community.md)  
**Sprint:** [`../../implementation-artifacts/sprint-status.yaml`](../../implementation-artifacts/sprint-status.yaml)
