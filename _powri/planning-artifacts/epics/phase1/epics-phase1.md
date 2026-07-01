---
stepsCompleted: [step-01-validate-prerequisites, step-02-design-epics, step-03-create-stories, step-04-final-validation]
inputDocuments:
  - _powri/planning-artifacts/prds/phase1/prd-phase1.md
  - project-context.md
  - docs/analytics/tracking-plan.md
  - content/content-model.md
phaseScope: Phase 1 (FR-1 through FR-5, FR-7 — FR-6 deferred to Phase 3)
---

# japan_winter_sport - Epic Breakdown

> **AI agents:** Do not load this entire file (~657 lines) for a single story.  
> **Start at [`INDEX.md`](../../INDEX.md)** — sharded epics in [`shards/`](shards/) + feature briefs in [`../../features/`](../../features/).  
> Example: Story 2.7 quiz → [`features/quiz/brief.md`](../../features/quiz/brief.md) + [`epics/phase1/shards/epic-02-discover.md`](shards/epic-02-discover.md#story-27).

## Overview

This document provides the complete epic and story breakdown for **Japan Ski & Snowboard Trip Planner** (Phase 1), decomposing requirements from PRD v1.1, embedded architecture (§7), UX specification (§12), and `docs/analytics/tracking-plan.md` into implementable stories.

**Scope:** Phase 1 only — all 20 resorts, bottom nav **Home · Explore · Info**, **no AI assistance**, no Plan tab, no live snow search, no accommodation map. **FR-6 (AI chat) + Story 3.6** deferred to **Phase 3 (monetisation gate)** — see [`epics/phase1/shards/phase-3-monetisation-ai.md`](shards/phase-3-monetisation-ai.md).

---

## Requirements Inventory

### Functional Requirements

FR-1: **Resort Discovery (Home & Explore)** — All 20 seed resorts render as browsable cards (hero image, name, region, tagline, badge summary). Filter chips (region, skill level, snow quality, distance) and search by name update the list without full page reload. Tapping a card navigates to resort detail (`resort_slug` in analytics). Explore tab reuses the same data with filter/quiz focus.

FR-2: **Resort Detail Page** — Hero image (static), quick verdict, highlights/lowlights, terrain data card, season-labelled T2 fields, link-out to official `trail_map_url`. Getting There reachable from detail scroll or dedicated section. Unsplash attribution on every hero image per §17.2.

FR-3: **Getting There** — Per resort: ≥2 transport options with duration/cost estimate and external booking links (clean outbound, §11.1). Gateway airport city shows ≥3 experience cards where applicable. Practical tips section (cash/card, onsen etiquette, JR Pass) collapsible and readable on mobile.

FR-4: **Find My Resort Quiz** — 4-question sliding-card quiz accessible from onboarding and Home/Explore. **Serious** (direct) and **Cosmic** (playful) question variants with toggle before Q1; both map to the same scoring enums and `score.ts`. **Results:** podium top 3 only (featured #1 + minimal #2/#3); ranks all 20 internally. **Cosmic results reveal:** interstitial + header + subline before podium (see `features/quiz/brief.md` § Results reveal). `quiz_started` and `quiz_completed` events fire per `docs/analytics/tracking-plan.md` (include `quiz_mode`, `result_count: 3`; `quiz_completed` after reveal completes).

FR-5: **Onboarding** — First-time visitors see ≤4 swipeable cards; skip and "Get started" both work. Onboarding not shown again after completion (local flag). No hero photography on onboarding cards.

FR-6: **AI Chat (Resort Q&A)** — **Phase 3 only** (deferred). Floating "Plan with AI →" on resort detail opens full-screen chat. Seed-content-first answers; §4.4 cost guardrails. Not in Phase 1 launch scope — gated on monetisation/sponsorship. See [`epics/phase1/shards/phase-3-monetisation-ai.md`](shards/phase-3-monetisation-ai.md).

FR-7: **Info** — Info tab accessible from bottom nav with links to Privacy Policy and Terms of Service (§17.1). About copy explains product purpose; app version visible. Contact/feedback method for data-rights requests (email sufficient for launch).

### NonFunctional Requirements

NFR-1: **Mobile-first responsive** — All screens designed for 375–430px viewport; bottom nav always visible (does not hide on scroll).

NFR-2: **Performance** — SSG/ISR for resort pages via Next.js build-time markdown import; onboarding and Info screens fast-load (no heavy photography). Home may paginate resort list for performance but all 20 reachable in ≤2 taps.

NFR-3: **Security (§6)** — Strict CSP, SRI on third-party assets, no inline scripts, HTTPS/HSTS, input sanitization on search and chat, API keys proxied server-side only, per-IP rate limiting (30 req/min search, 10 req/min AI chat), CORS restricted to own domain.

NFR-4: **Privacy & consent (§6, §13, §17)** — GDPR/PDPA cookie consent banner; anonymous/cookieless analytics by default; no PII in event properties; `consent_state` on every event.

NFR-5: **Content pipeline (§11.7)** — Markdown in `content/resorts/<slug>.md` is source of truth; imported at build time; filter by `content_status: published`; no headless CMS in Phase 1.

NFR-6: **Season currency (§4.3)** — T2 fields display explicit season labels (e.g. "2025/26 season"); UI surfaces season label and last-reviewed signal; never present stale numbers as current.

NFR-7: **Unsplash attribution (§17.2)** — Photographer name + profile link + Unsplash link with UTM params on every image; reusable attribution component; download endpoint triggered on first use.

NFR-8: **Analytics from launch (§13)** — Instrument all Phase 1 events in `docs/analytics/tracking-plan.md`; verify in PostHog live-events view before feature sign-off.

NFR-9: **AI cost guardrails (§4.4)** — Max 10 user turns/session, ~1000 token input cap, 600–800 max output tokens, ~6-turn context cap, 30 messages or 5 sessions/day per IP, global daily spend kill-switch with non-AI fallback.

NFR-10: **Accessibility baseline** — Sufficient colour contrast on design tokens; keyboard/tap targets ≥44px on interactive controls; semantic HTML for nav and sections.

NFR-11: **Dependency hygiene (§6.6)** — `npm audit` clean of high/critical vulns; lock file committed.

NFR-12: **Clean outbound links (§11.1)** — No affiliate tracking; external links open safely (`rel="noopener noreferrer"`).

NFR-13: **Counter-metrics instrumentation (§2)** — Track home bounce proxy, `filter_zero_results`, external-link CTR, satisfaction micro-survey after 2nd distinct resort detail view per session.

NFR-14: **English-only Phase 1** — i18n infrastructure scaffolded but only EN ships; locale property on events defaults to `en`.

### Additional Requirements

- **Starter stack:** Next.js (React) mobile-first, Tailwind CSS + design tokens (§12.2), Leaflet + OSM default for maps (Phase 1 minimal usage), Zustand or React Context for client state.
- **Markdown loader:** `gray-matter` (or equivalent) + content loader reading YAML frontmatter per `content/content-model.md` §8.1 schema including `images:` block.
- **Serverless AI proxy:** Vercel Edge Functions or AWS Lambda proxying Anthropic Claude API; never expose API key client-side.
- **Seed-content-first AI (§4.1):** Retrieve relevant resort fields before model call; prompt caching for static system prompt + per-resort context; Haiku for routing/simple FAQ, Sonnet for synthesis.
- **Analytics tooling:** PostHog (EU Cloud) + Plausible; feature flags scaffold from Phase 1.
- **Home hero images:** Pre-selected in `content/site-images.md`; carousel crossfade per §12.5 Screen 1.
- **Shared-element transition:** Resort card tap → detail hero (§12.5).
- **Legal pages:** Privacy Policy + Terms of Service required before Ring 2 public sharing; draft for Phase 1 launch gate.
- **No monetisation Phase 1:** No affiliate code, booking widgets, or ad infrastructure.
- **Content gate:** All 20 resorts must be `content_status: published` before launch (§10.2 content-freeze gate).
- **Image service abstraction (R1):** Abstract Unsplash behind internal image interface for provider swap.

### UX Design Requirements

UX-DR1: Implement **Default Theme (Theme A)** design tokens — colours (`--color-bg` through `--color-nav-bg`), 8px spacing grid, card/button/badge radii per §12.2 Theme A table.

UX-DR2: Load and apply **Default Theme typography** — Cormorant Garamond (display) + DM Sans (body/UI) from Google Fonts per §12.3 token sizes and weights.

UX-DR3: Build **`BottomNav`** component — 3 tabs (Home · Explore · Info), always visible, active state with accent underline or dot, DM Sans 11px labels.

UX-DR4: Build **`HeroCarousel`** — full-bleed crossfade carousel, auto-advance 3–4s hold / 1s fade, no slide animation, white gradient overlay for tagline.

UX-DR5: Build **`ResortCard`** — full-width image, name, region tag, 2–3 badge pills, chevron; supports shared-element transition to detail hero.

UX-DR6: Build **`FilterChip`** — single-select, active/inactive states, instant filter without page reload.

UX-DR7: Build **`QuickVerdict`** — italic serif pull quote overlaid on detail hero with dark gradient backing.

UX-DR8: Build **`HighlightLowlightRow`** — 50/50 two-column layout, blue "Highlights" / amber "Watch out for" headers, tinted row backgrounds.

UX-DR9: Build **`StatRow`** — horizontal row of 3–4 quick stat chips (runs, vertical, snowfall, season).

UX-DR10: Build **`TransportRow`** — icon + method + duration + cost + external link chip.

UX-DR11: Build **`OnboardingCard`** — full-screen swipeable card, icon + title + body, progress dots, skip top-right, no photographs.

UX-DR12: Build **`ResortQuizCard`** — full-screen sliding quiz card, wide full-width tappable answer buttons, 4 questions max. Support **Serious / Cosmic** copy variants via shared enum options.

UX-DR13: Build **`ChatBubble`** — user (accent blue, right) and AI (white card, border, left) variants; typing indicator (three dots).

UX-DR14: Build **`UnsplashAttribution`** — reusable component per §17.2 HTML template with UTM params.

UX-DR15: Build **`SectionHeader`** — small uppercase DM Sans label + optional "See all →" link with divider rule.

UX-DR16: Sticky top bar on Home — transparent over hero, transitions to white + 1px border on scroll past hero; search icon expands inline search bar.

UX-DR17: Floating **"Plan with AI →"** CTA on resort detail — accent blue, full-width, 52px height, pinned above bottom nav.

UX-DR18: Info screen — minimal layout, no photography, fast load.

### FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR-1 | Epic 2 | Home & Explore resort discovery, filters, search |
| FR-4 | Epic 2 | Find My Resort quiz |
| FR-5 | Epic 2 | First-time onboarding |
| FR-2 | Epic 3 | Resort detail page sections & attribution |
| FR-3 | Epic 3 | Getting There transport, gateway, tips |
| FR-6 | **Phase 3** | AI resort Q&A — Epic 4 + Story 3.6 (deferred) |
| FR-7 | Epic 5 | Info tab, legal links, contact, version |

**Cross-cutting (Phase 1 epics):** NFR-3–5, NFR-8, UX-DR1–2 enforced from Epic 1 onward; NFR-7 in Epic 3; NFR-13 in Epic 5. **NFR-9 (AI guardrails)** applies when Phase 3 ships.

---

## Epic List

### Epic 1: App Foundation & Content Pipeline
Establish the Next.js app shell, design system, markdown content pipeline, analytics/consent foundation, and bottom navigation so all feature epics can ship on a consistent, secure base.
**FRs covered:** (enables all) — NFR-3, NFR-4, NFR-5, NFR-8, NFR-11, NFR-14, UX-DR1–3

### Epic 2: Discover Your Resort
First-time users complete onboarding, browse all 20 resorts on Home and Explore, filter and search instantly, and take the Find My Resort quiz to get personalised recommendations.
**FRs covered:** FR-1, FR-4, FR-5

### Epic 3: Resort Detail & Getting There *(Phase 1 complete at Story 3.5)*
Users open any resort and get a photography-rich detail page with honest highlights/lowlights, terrain stats, season-labelled data, trail map link-out, and complete Getting There guidance.
**FRs covered:** FR-2, FR-3

### Epic 4: AI Resort Assistant *(Phase 3 — deferred)*
Users ask evergreen questions about a resort via full-screen chat powered by seed content first, with bounded API costs and graceful degradation.
**FRs covered:** FR-6 · **Gate:** monetisation/sponsorship · **See:** [`epics/phase1/shards/phase-3-monetisation-ai.md`](shards/phase-3-monetisation-ai.md)

### Epic 5: Info, Legal & Launch Readiness
Users access About, legal pages, and contact from the Info tab; satisfaction micro-survey and remaining analytics complete Phase 1 launch gates.
**FRs covered:** FR-7 — NFR-13

---

## Epic 1: App Foundation & Content Pipeline

Users and developers have a secure, styled, analytics-ready app shell that loads all 20 published resorts from markdown at build time.

### Story 1.1: Initialize Next.js Project & Deployment Baseline

As a developer,
I want a Next.js mobile-first project with Tailwind, TypeScript, and Vercel deployment config,
So that feature work starts on the approved stack with CI-ready builds.

**Acceptance Criteria:**

**Given** an empty app codebase
**When** the project is initialized per PRD §7.1
**Then** Next.js (App Router), TypeScript, Tailwind CSS, and ESLint are configured
**And** `package-lock.json` is committed and `npm audit` reports no high/critical vulnerabilities (NFR-11)
**And** HTTPS/HSTS headers are configured for production deployment (NFR-3)
**And** a health-check route returns 200 on deploy smoke test

### Story 1.2: Design Tokens & Default Theme Typography

As a user,
I want the app to look like a premium travel magazine with consistent colours and type,
So that every screen feels cohesive and trustworthy.

**Acceptance Criteria:**

**Given** Theme A (Default) tokens in PRD §12.2–12.3
**When** any page renders
**Then** CSS custom properties implement all Default Theme colour tokens (UX-DR1)
**And** Cormorant Garamond and DM Sans load from Google Fonts with specified weights/sizes (UX-DR2)
**And** spacing follows 8px grid; card, button, and badge radii match Theme A table
**And** body line-height is 1.6; display headings 1.15

### Story 1.3: Markdown Content Loader & Resort Types

As a developer,
I want resort markdown imported at build time with typed frontmatter,
So that all features read from a single content source of truth.

**Acceptance Criteria:**

**Given** files in `content/resorts/<slug>.md` following `content-model.md` schema
**When** the build runs
**Then** a content loader parses YAML frontmatter + markdown body via gray-matter (or equivalent) (NFR-5)
**And** only resorts with `content_status: published` are included in the build output
**And** TypeScript types cover required §8.1 fields including `images:`, `trail_map_url`, transport, gateway experiences, and season-labelled T2 fields
**And** a dev-time warning logs unpublished slugs without breaking the build

### Story 1.4: App Shell, Routing & Bottom Navigation

As a user,
I want persistent bottom navigation between Home, Explore, and Info,
So that I can move around the app without losing context.

**Acceptance Criteria:**

**Given** the app is open on any main tab
**When** I view the screen on a 375–430px viewport
**Then** `BottomNav` shows Home · Explore · Info with active-state indicator (UX-DR3, NFR-1)
**And** the nav bar remains visible and does not hide on scroll (§12.5)
**And** routes exist for `/`, `/explore`, `/info`, and `/resorts/[slug]` (placeholder pages acceptable until Epic 2–3)
**And** mobile-first layout uses warm off-white `--color-bg` page background

### Story 1.5: Analytics, Consent Banner & Common Event Properties

As a product owner,
I want privacy-first analytics with consent gating from day one,
So that we can measure funnels without violating GDPR/PDPA.

**Acceptance Criteria:**

**Given** a first-time visitor with no consent stored
**When** the app loads
**Then** PostHog (EU) and Plausible initialize in anonymous/cookieless mode (NFR-4, NFR-8)
**And** a cookie/consent banner gates identified/persistent tracking until accepted
**And** a shared analytics helper attaches common properties (`locale`, `device_type`, `is_logged_in`, `consent_state`, `app_version`) per `docs/analytics/tracking-plan.md` §1
**And** `$pageview` captures automatically; helper supports explicit event firing used by later stories
**And** no PII is sent in event properties

### Story 1.6: Security Headers & API Route Scaffold

As a developer,
I want CSP, CORS, and serverless route scaffolding in place,
So that later AI and search endpoints are secure by default.

**Acceptance Criteria:**

**Given** the production build
**When** security headers are inspected
**Then** strict CSP allows scripts only from trusted sources with no inline scripts/eval (NFR-3)
**And** third-party scripts use SRI where loaded from CDN
**And** an `/api/` route namespace exists with CORS restricted to own domain
**And** rate-limiting middleware scaffold is ready for AI (10 req/min) and search (30 req/min) endpoints

### Story 1.7: i18n Scaffold (English Only)

As a developer,
I want i18n infrastructure ready for Phase 2 languages,
So that EN ships now without rework later.

**Acceptance Criteria:**

**Given** Phase 1 English-only scope (NFR-14)
**When** the app renders
**Then** all user-facing strings route through an i18n layer defaulting to `en`
**And** `locale` property on analytics events defaults to `en`
**And** no TH/JP copy ships in Phase 1 UI

---

## Epic 2: Discover Your Resort

Users complete onboarding once, browse and filter all 20 resorts on Home and Explore, search by name, and take the quiz for personalised recommendations.

### Story 2.1: First-Time Onboarding Flow

As a first-time visitor,
I want a brief swipeable introduction to the app,
So that I understand what the product offers before browsing resorts.

**Acceptance Criteria:**

**Given** I have not completed onboarding (`localStorage` or equivalent flag unset)
**When** I open the app
**Then** I see ≤4 full-screen `OnboardingCard` slides with icon, title, and body — no photographs (FR-5, UX-DR11)
**And** progress dots, skip (top-right), and "Get started" on the final card all work
**And** completing or skipping sets a persistent flag so onboarding never shows again
**And** final card offers entry to Find My Resort quiz (links to Story 2.6)

### Story 2.2: Home Hero Carousel & Tagline

As a user landing on Home,
I want a stunning full-bleed hero with curated Japan ski imagery,
So that the first impression matches the premium travel-magazine vision.

**Acceptance Criteria:**

**Given** `content/site-images.md` defines home hero assets
**When** I open Home
**Then** `HeroCarousel` displays crossfade slides (3–4s hold, 1s fade, no slide animation) (UX-DR4, §12.5 Screen 1)
**And** white gradient overlay and tagline in Cormorant Garamond 600 appear over the hero
**And** `UnsplashAttribution` renders on hero images per §17.2 (UX-DR14)
**And** sticky top bar is transparent over hero and transitions to white + border on scroll (UX-DR16)

### Story 2.3: Resort List & Resort Cards (All 20)

As a user,
I want to scroll through all 20 Japan ski destinations as beautiful cards,
So that I can browse every option from one screen.

**Acceptance Criteria:**

**Given** 20 published resort content files
**When** I view Home below the filter chips
**Then** each resort renders as a `ResortCard` with hero image, name, region, tagline, and 2–3 badge pills (FR-1, UX-DR5)
**And** all 20 resorts are reachable — initial render may show 10 with "Show more" loading remainder inline; ≤2 taps to see all (§12.5)
**And** tapping a card navigates to `/resorts/[slug]` and fires `resort_card_tapped` with `resort_slug`, `position`, `list_context: home`
**And** shared-element transition is initiated from card image to detail hero (UX-DR5, §12.5)

### Story 2.4: Filter Chips & Instant List Filtering

As a user,
I want to filter resorts by region and attributes instantly,
So that I can narrow 20 destinations without waiting for page loads.

**Acceptance Criteria:**

**Given** the Home or Explore resort list
**When** I tap a filter chip (All · Hokkaido · Nagano · Niigata · Tohoku · Beginner Friendly · Deep Powder · Family)
**Then** the list filters client-side with no full page reload (FR-1, UX-DR6)
**And** single-select behaviour applies; active chip shows active state
**And** `filter_applied` fires with `filter_type`, `filter_value`, `results_count`
**And** if zero results, `filter_zero_results` fires (NFR-13 guardrail)

### Story 2.5: Inline Search by Resort Name

As a user,
I want to search resorts by name from the top bar,
So that I can jump directly to a destination I already have in mind.

**Acceptance Criteria:**

**Given** Home sticky top bar with search icon
**When** I tap the search icon
**Then** an inline search bar slides down with smooth animation (UX-DR16)
**And** results update live as I type, matching resort names
**And** `search_performed` fires with `results_count` (query text only if `consent_state=granted` and scrubbed)
**And** selecting a result navigates to resort detail with `resort_card_tapped` or equivalent navigation event

### Story 2.6: Explore Tab

As a user who wants to compare options,
I want an Explore tab that foregrounds filters and the quiz,
So that discovery feels purposeful rather than passive browsing.

**Acceptance Criteria:**

**Given** I tap Explore in bottom nav
**When** the Explore screen opens
**Then** filter chips and "Find my resort" CTA are prominent at top (FR-1, §12.5)
**And** the same 20-resort dataset and card components as Home are reused — not a separate data source
**And** list events use `list_context: filtered` or `explore` as appropriate
**And** default scroll/focus differs from Home (filters visible without scrolling past hero)

### Story 2.7: Find My Resort Quiz

As an undecided planner,
I want a short 4-question quiz that recommends resorts,
So that I get a personalised starting list without reading all 20.

**Acceptance Criteria:**

**Given** entry from onboarding final card, Home, or Explore CTA
**When** I start the quiz
**Then** `quiz_started` fires with `entry_point` and `quiz_mode` (`serious` default, or `cosmic`)
**And** I can toggle **Serious / Cosmic** before the first question; both variants use wide full-width answer buttons (FR-4, UX-DR12)
**And** four `ResortQuizCard` screens show the active variant's question copy per §12.5 / [`features/quiz/brief.md`](../../features/quiz/brief.md)
**And** each selected answer maps to shared scoring enums per [`features/quiz/scoring.md`](../../features/quiz/scoring.md) — single `score.ts` for both modes
**And** swiping or tapping advances to the next question
**And** on completion, a **podium of top 3 resorts** appears with mode-specific results reveal per [`features/quiz/brief.md`](../../features/quiz/brief.md) § Results reveal + § Results podium:
  - **Serious:** header only (*"Based on your answers…"*) → featured #1 + minimal #2/#3
  - **Cosmic:** ~800ms interstitial (*"Consulting the powder oracle…"*), then header + subline (*"The stars have spoken…"* / *"Ranked by cosmic vibes…"*), then podium
  - **#1 featured:** accent border, rank **"1"** + label, taller hero, full card detail, **match reason** one-liner
  - **#2/#3 minimal:** 48px thumbnail, name, region, rank badge — no tagline/badges
  - **Browse CTA:** *"Browse all resorts →"* → Explore (full list unchanged there)
  - **No answer summary** tags under header; `score.ts` still ranks all 20 internally
**And** `quiz_completed` fires after Cosmic interstitial completes (when results header is visible)
**And** `quiz_completed` fires with enum values, `quiz_mode`, and **`result_count: 3`**
**And** tapping a podium result navigates to detail with `list_context: quiz` and `position` 1 / 2 / 3

---

## Epic 3: Resort Detail & Getting There

Users open any resort and receive honest, photography-rich detail plus complete travel guidance including transport, gateway city experiences, and practical tips.

### Story 3.1: Resort Detail Layout, Hero & Quick Verdict

As a user who tapped a resort card,
I want a seamless hero transition and an immediate editorial verdict,
So that I feel confident I'm viewing the right destination.

**Acceptance Criteria:**

**Given** I navigate to `/resorts/[slug]`
**When** the detail page loads
**Then** full-bleed hero (55vh) uses the same image as the list card with shared-element transition (FR-2, §12.5 Screen 2)
**And** resort name (Cormorant Garamond 600), region pill, and back arrow overlay the hero
**And** `QuickVerdict` italic pull quote sits on lower hero with gradient backing (UX-DR7)
**And** `resort_detail_viewed` fires with `resort_slug` and `source`
**And** `UnsplashAttribution` is visible on the hero (NFR-7, UX-DR14)

### Story 3.2: Season Snapshot, Highlights, Lowlights & Terrain

As a user researching a resort,
I want scannable pros/cons and mountain stats with honest season labels,
So that I can judge fit without hype.

**Acceptance Criteria:**

**Given** a published resort content file
**When** I scroll the detail page
**Then** section order is: Season snapshot → Highlights & Lowlights → Terrain & Mountain Data → Trail map link → Getting There (FR-2, §12.5)
**And** `HighlightLowlightRow` shows two 50/50 columns with tinted backgrounds (UX-DR8)
**And** `StatRow` displays runs, vertical, area, lifts, and typical season (UX-DR9)
**And** T2 fields show explicit season labels e.g. "¥X — 2025/26 season" and last-reviewed signal (NFR-6)
**And** each section uses `SectionHeader` with divider rule (UX-DR15)

### Story 3.3: Trail Map Link-Out

As a user who wants terrain detail,
I want a clear link to the official trail map,
So that I can study runs on the resort's authoritative map.

**Acceptance Criteria:**

**Given** a resort with `trail_map_url` in frontmatter
**When** I tap "View trail map" (or equivalent) on the detail page
**Then** the official trail map opens in a new tab with `rel="noopener noreferrer"` (FR-2, NFR-12)
**And** `trail_map_opened` fires with `resort_slug`
**And** `external_link_clicked` fires with `link_type: trail_map` and `destination_domain`

### Story 3.4: Getting There — Transport Options

As a user planning logistics,
I want clear transport options with time, cost, and booking links,
So that I know how to reach the resort from major gateways.

**Acceptance Criteria:**

**Given** a resort with ≥2 transport options in content
**When** I view Getting There (inline section or dedicated route `/resorts/[slug]/getting-there`)
**Then** each option renders as a `TransportRow` with icon, method, duration, and cost range (FR-3, UX-DR10). **Phase 1:** no booking/monetisation chips — editorial notes only; booking link-outs are a post-monetisation phase (PRD §11.1).
**And** `getting_there_viewed` fires with `resort_slug`

### Story 3.5: Gateway City Experiences & Practical Tips

As a user flying into Japan,
I want gateway city highlights and collapsible practical tips,
So that I can plan arrival days and avoid cultural pitfalls.

**Acceptance Criteria:**

**Given** a resort with gateway airport city content
**When** I scroll Getting There
**Then** ≥3 experience cards display (image + name + 1-line description) with "See more" collapse if >3 (FR-3)
**And** practical tips (cash vs card, onsen etiquette, JR Pass, IC Card, language basics) render as collapsible FAQ rows (FR-3, §12.5 Screen 4)
**And** tips are readable on mobile without horizontal scroll
**And** experience card images include `UnsplashAttribution` where applicable

### Story 3.6: Floating "Plan with AI" Entry Point — **DEFERRED → Phase 3**

> **Not Phase 1.** Moved to [`epics/phase1/shards/phase-3-monetisation-ai.md`](shards/phase-3-monetisation-ai.md) with Epic 4. Implement after monetisation/sponsorship gate — AI is high ongoing cost.

As a user with questions about this resort,
I want an obvious AI entry point on the detail page,
So that I can ask follow-up questions without hunting for a menu item.

**Acceptance Criteria:** *(unchanged — execute in Phase 3)*

**Given** I am on resort detail
**When** I view the bottom of the screen
**Then** floating "Plan with AI →" button appears above bottom nav — accent blue, full-width, 52px (UX-DR17, FR-6 entry)
**And** tapping it opens the AI chat overlay (Epic 4)
**And** bottom nav remains visible beneath the button offset

---

## Epic 4: AI Resort Assistant — **DEFERRED → Phase 3**

> **Not Phase 1.** Full epic deferred until sponsorship/monetisation phase. See [`epics/phase1/shards/phase-3-monetisation-ai.md`](shards/phase-3-monetisation-ai.md).

### Story 4.1: AI Chat UI Overlay

As a user on a resort detail page,
I want a full-screen conversational interface,
So that I can ask questions naturally on mobile.

**Acceptance Criteria:**

**Given** I tap "Plan with AI →" on resort detail
**When** the chat opens
**Then** a full-screen overlay slides up from bottom with close (×) top-right (FR-6, §12.5 Screen 5)
**And** AI first message displays the warm opener from §12.5
**And** `ChatBubble` renders user messages right (accent blue) and AI messages left (white card) (UX-DR13)
**And** input bar is pinned bottom and keyboard-aware
**And** typing indicator (three animated dots) shows while awaiting response
**And** `ai_chat_opened` fires with `entry_point: floating_button`

### Story 4.2: Seed-Content Retrieval & AI Proxy

As a user asking about a resort,
I want answers grounded in the same content I see on the page,
So that responses are accurate and consistent.

**Acceptance Criteria:**

**Given** an authenticated serverless `/api/chat` route (API key server-side only, NFR-3)
**When** I send a message about the current resort
**Then** the handler retrieves relevant fields from the resort markdown (§4.1 seed-content-first) before calling Claude
**And** prompt caching applies to static system prompt + per-resort context block (§4.4.3)
**And** Haiku classifies intent/routes simple FAQ; Sonnet handles synthesis (§4.4.3, §7.4)
**And** live web search is NOT invoked in Phase 1 (FR-6)
**And** `ai_chat_message_sent` fires with `message_index`
**And** user input is sanitized; no raw HTML rendered from user messages (NFR-3)

### Story 4.3: AI Cost Guardrails & Fallback

As a product owner,
I want bounded AI spend with graceful degradation,
So that free chat cannot bankrupt the product.

**Acceptance Criteria:**

**Given** §4.4.1 limits
**When** a user exceeds 10 messages in a session
**Then** a friendly message offers to start fresh / summarise; context resets (NFR-9)
**And** messages over ~1000 tokens are trimmed with a friendly prompt to shorten
**And** API `max_tokens` is capped at 600–800 per response
**And** context includes last ~6 turns or rolling summary only
**And** per-IP limit of ~30 messages or 5 sessions/day returns a graceful rate-limit message
**And** global daily spend kill-switch degrades to non-AI fallback (canned answers + seed-content search) without unbounded spend (NFR-9)
**And** rate limiting on `/api/chat` enforces 10 req/min per IP (NFR-3)

### Story 4.4: Pre-Generated FAQ Cache (Optional Fast Path)

As a user asking common questions,
I want instant answers to frequent resort FAQs,
So that common queries cost zero API tokens.

**Acceptance Criteria:**

**Given** top recurring questions identified per resort (e.g. beginner-friendliness, best time to visit, transport from CTS)
**When** user intent matches a cached FAQ
**Then** the answer serves from pre-generated seed-content responses with no live Claude call (§4.4.2)
**And** fallback to Story 4.2 occurs for unmatched intents
**And** cached path is indistinguishable in UX from live responses

---

## Epic 5: Info, Legal & Launch Readiness

Users access About, legal, and contact information; satisfaction feedback and launch gates complete Phase 1.

### Story 5.1: Info Tab — About, Contact & Version

As a user,
I want an Info tab with product context and a way to reach the team,
So that I understand what the app is and can exercise my data rights.

**Acceptance Criteria:**

**Given** I tap Info in bottom nav
**When** the Info screen loads
**Then** About section explains product purpose in 2–3 sentences (FR-7, UX-DR18)
**And** Contact shows email for feedback and GDPR/PDPA data-rights requests
**And** app version string displays at bottom
**And** layout is minimal with no photography for fast load

### Story 5.2: Privacy Policy & Terms of Service Pages

As a user and legal stakeholder,
I want accessible Privacy Policy and Terms pages linked from Info,
So that the product meets GDPR/PDPA before public sharing.

**Acceptance Criteria:**

**Given** Info tab legal section
**When** I tap Privacy Policy or Terms of Service
**Then** dedicated pages render at `/privacy` and `/terms` (FR-7, §17.1)
**And** Privacy Policy covers analytics (§13), third parties (Anthropic, PostHog, Plausible, Unsplash), consent, retention, and user rights
**And** Terms include "information as-is, verify against official resort sources" disclaimer (§17.1, R5)
**And** pages are readable on mobile without external dependencies

### Story 5.3: Satisfaction Micro-Survey

As a product owner,
I want lightweight satisfaction signal after meaningful browsing,
So that we can track the ≥4.2/5 confidence goal.

**Acceptance Criteria:**

**Given** a session where the user viewed a 2nd distinct resort detail page
**When** the second detail view completes
**Then** a once-per-session prompt asks "How helpful is this for planning your trip?" (1–5 stars) (§2, NFR-13)
**And** submitting fires `satisfaction_prompt_submitted` with `score` and `resort_views_before_prompt`
**And** dismissing does not re-prompt in the same session
**And** prompt does not block core navigation

### Story 5.4: Phase 1 Analytics Verification & Tracking Plan Sync

As a developer,
I want every Phase 1 event verified in PostHog,
So that launch meets the implementation checkpoint gate.

**Acceptance Criteria:**

**Given** all Phase 1 features implemented
**When** a test pass runs through onboarding → quiz → filter → detail → trail map → getting there → Info
**Then** every Phase 1 event in `docs/analytics/tracking-plan.md` §2 fires with correct properties (NFR-8)
**And** common properties attach where relevant
**And** any new/changed events are reflected in `docs/analytics/tracking-plan.md` in the same change
**And** events respect `consent_state` gating rules

### Story 5.5: Launch Gate Checklist (Content & QA)

As the product owner,
I want a documented pass against §10.2 release gates,
So that Phase 1 ships only when quality criteria are met.

**Acceptance Criteria:**

**Given** all Phase 1 stories complete (Epics 1–3 Stories 3.1–3.5, Epic 5)
**When** the launch checklist is executed
**Then** all 20 resorts have `content_status: published` and correct season labels (§10.2 content-freeze)
**And** cross-device responsive pass on 375px and 430px widths (NFR-1)
**And** broken-link check passes for all external URLs (trail maps, transport, Unsplash)
**And** Unsplash attribution visible on every image site-wide (NFR-7)
**And** home bounce, filter-zero, and external-link CTR baselines are observable in PostHog (NFR-13)
**And** smoke test passes on production deploy

---

## Final Validation Summary

| Check | Status |
|-------|--------|
| All FR-1…FR-7 mapped to epics | ✅ (FR-6 → Phase 3) |
| All FRs covered by ≥1 story with testable ACs | ✅ |
| NFRs distributed across stories | ✅ |
| UX-DR1–18 covered (Epic 1–5) | ✅ |
| No forward story dependencies within epics | ✅ |
| Epics deliver standalone user value in sequence | ✅ |
| No "create all tables upfront" anti-pattern | ✅ (markdown build-time, no DB Phase 1) |
| Starter template → Epic 1 Story 1.1 | ✅ |
| Architecture from PRD §7 (no separate Architecture.md) | ✅ Noted in Additional Requirements |
| Phase 2+ explicitly out of scope | ✅ |

**Story count:** 27 stories across 5 epics  
**Recommended implementation order (Phase 1):** Epic 1 → 2 → 3 (Stories 3.1–3.5) → 5. **Phase 3 (post-monetisation):** Story 3.6 → Epic 4.

*Generated from PRD v1.1 (approved 2026-06-12) · Phase 1 scope updated 2026-06-13 — FR-6/AI deferred*
