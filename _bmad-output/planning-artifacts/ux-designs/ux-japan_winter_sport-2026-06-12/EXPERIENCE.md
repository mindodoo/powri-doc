---
name: Japan Ski Trip Planner
status: final
updated: 2026-06-12
sources:
  - japan_ski_prd.md#12-uiux-specification
  - japan_ski_prd.md#3-core-features
  - _bmad-output/planning-artifacts/epics.md
  - docs/tracking-plan.md
---

# Japan Ski Trip Planner — Experience Spine

> Phase 1 scope. Visual identity: `DESIGN.md` (Theme A Default). Token references use `{path.to.token}` syntax. **Spines win on conflict** with mocks, wireframes, or imports.

## Foundation

**Form factor:** Mobile-first responsive web app. Primary viewport 375–430px (iPhone SE → Pro Max). Desktop is not a Phase 1 design target — layout may degrade gracefully but is not optimised.

**UI system:** Tailwind CSS + custom design tokens from `DESIGN.md`. No shadcn/MUI — bespoke editorial components. Icons: Lucide line set (§12.9).

**Phase boundary:** IA and flows below cover Phase 1 only. Accommodation map (Screen 3), Plan tab, comparison, and live snow search are Phase 2–3 — listed as future surfaces, not designed here.

**Analytics:** Instrument per `docs/tracking-plan.md` §2. Every explicit event respects `consent_state`.

## Information Architecture

| Surface | Route / entry | Phase | Purpose |
|---------|---------------|-------|---------|
| Onboarding | First visit overlay | 1 | Orient new users; optional quiz entry |
| Find My Resort (quiz) | `/quiz` or modal stack from Home/Explore/Onboarding | 1 | 4-question preference quiz → filtered list |
| Home | `/` · BottomNav | 1 | Hero carousel + filter chips + resort list |
| Explore | `/explore` · BottomNav | 1 | Same data as Home; filters + quiz CTA prominent |
| Resort Detail | `/resorts/[slug]` | 1 | Editorial resort page + AI entry |
| Getting There | Detail section or `/resorts/[slug]/getting-there` | 1 | Transport, gateway city, practical tips |
| AI Chat | Full-screen overlay from detail | 1 | Seed-content resort Q&A |
| Info | `/info` · BottomNav | 1 | About, legal, contact, version |
| Privacy / Terms | `/privacy`, `/terms` | 1 | Legal pages linked from Info |
| Accommodation & Map | Detail section | 2 | Deferred |
| Plan / My Trip | BottomNav tab | 3 | Deferred |

**Navigation model:** Bottom tab bar — **Home · Explore · Info** — always visible, never hide-on-scroll. Modal depth: one level (quiz, AI chat, search expand). Back from detail returns to prior list context preserving scroll where feasible.

→ Home reference: `mockups/key-home.html`  
→ Detail reference: `mockups/key-resort-detail.html`  
→ Quiz reference: `mockups/key-quiz.html`, `mockups/key-quiz-results.html`

## Inspiration & Anti-patterns

**Embrace (from PRD §12.0)**
- Kinfolk: breathing room, warm off-white, photography as hero
- Hokuoh: full-width image → understated text stack
- Monocle: clear section headers, geographic tagging, confident copy
- Airbnb: instant filter chips, card-forward lists, bottom nav persistence

**Reject**
- Booking-engine urgency ("Only 2 left!", countdown timers)
- Gamification (streaks, points, badges for engagement)
- Horizontal side-by-side comparison on mobile (Phase 3 uses stacked rows instead)
- Hero video loops, autoplay audio, parallax scroll gimmicks
- Illustrated mascots or large decorative art replacing photography
- Hide-on-scroll navigation
- Generic "Sign up to unlock" gating (no accounts Phase 1)

## Voice and Tone

Microcopy and AI dialogue. Brand posture lives in `DESIGN.md` Brand & Style.

| Do | Don't |
|----|-------|
| "Based on your answers, here are resorts we'd recommend." | "Perfect match found!" |
| "Watch out for" (lowlights header) | "Cons" or "Problems" |
| "2025/26 season" on lift-pass figures | Unlabelled prices that imply live rates |
| "Let's start a fresh chat to keep things snappy." (AI limit) | "Error: token limit exceeded" |
| Warm, complete sentences | Exclamation marks, hype, superlatives without evidence |
| Honest lowlights | Burying negatives or weasel words |

**AI first message (Phase 1):** *"Hey! I'm here to help you put together your Japan ski trip. Tell me a bit about yourself — your level on the mountain, who you're going with, and roughly when you're thinking of going. We can go from there."*

## Component Patterns

Behavioral rules. Visual specs in `DESIGN.md` Components.

| Component | Use | Behavioral rules |
|-----------|-----|------------------|
| BottomNav | All main surfaces | 3 tabs Phase 1. Tap switches route; active state persists. Never hidden. |
| HeroCarousel | Home | Auto crossfade; no user controls. Attribution always visible. |
| ResortCard | Home, Explore, search | Whole card tappable → detail with shared-element hero transition. Fires `resort_card_tapped`. **Not used on quiz results** (see podium components). |
| QuizTopMatchCard | Quiz results | Featured #1 — accent border, dual labels, taller hero, match reason, full detail. `position: 1`. |
| QuizRunnerUpRow | Quiz results | Minimal #2/#3 — 48px thumb, name, region, rank badge. `position: 2` / `3`. |
| FilterChip | Home, Explore | Single-select. Instant client filter; no navigation. Fires `filter_applied` / `filter_zero_results`. |
| Search expand | Home, Explore top bar | Icon tap slides bar down; live filter by name. Fires `search_performed`. |
| OnboardingCard | First visit | ≤4 cards; skip or complete sets persistent flag; no return on subsequent visits. |
| ResortQuizCard | Quiz flow | 4 questions max; wide answer buttons; slide-right advance 200ms. |
| QuickVerdict | Detail hero | Static on load; not editable. |
| HighlightLowlightRow | Detail | Scrolls with page; equal column weight. |
| StatRow | Detail | Season snapshot + terrain stats; season label on T2 values. |
| TransportRow | Getting There | External link opens new tab; fires `external_link_clicked`. |
| CTA Floating | Detail | Opens AI overlay; remains above BottomNav. |
| ChatBubble | AI chat | User sends on submit; AI streams or shows typing indicator. Input sanitised. |
| UnsplashAttribution | All images | Required on every photo; links open Unsplash with UTM. |
| SectionHeader | Detail sections | Dividers between major blocks. |
| Satisfaction prompt | After 2nd detail view / session | Once per session; 1–5 stars; dismissible. |

**Phase 2 placeholders (do not build Phase 1):** AccommodationCard, MapPin, MapBottomSheet, ResortCardFeatured.

## State Patterns

| State | Surface | Treatment |
|-------|---------|-----------|
| First visit | App | Show Onboarding; if skipped, land Home. |
| Returning visit | App | Skip onboarding; land Home or last tab. |
| Home loading | Home | 3–4 ResortCard skeleton shimmers; hero may load progressively. |
| Filter active | Home / Explore | Chips reflect selection; list filtered; count in analytics. |
| Filter empty | Home / Explore | Line-art mountain icon + "No resorts match — try a different filter." |
| Search active | Home / Explore | Inline bar expanded; list live-filters; empty → same as filter empty. |
| Quiz in progress | Quiz | Progress implied by question index; back discards or confirms [ASSUMPTION: no back — forward only]. |
| Quiz complete | Quiz → results reveal → podium | Serious: header → top 3 podium. Cosmic: interstitial → header + subline → podium; Browse CTA → Explore; `quiz_completed` after reveal visible (`result_count: 3`) |
| Detail loading | Detail | Hero skeleton + content shimmer below fold. |
| Detail loaded | Detail | Full section stack per IA; floating AI CTA visible. |
| Getting There expanded tip | Getting There | FAQ row tap expands one section; accordion-style one-at-a-time [ASSUMPTION]. |
| AI chat open | Overlay | Previous screen retained underneath; close returns to detail scroll position. |
| AI typing | AI chat | Three-dot indicator; input disabled briefly. |
| AI limit reached | AI chat | Friendly cap message; offer fresh chat. Non-AI fallback if global kill-switch. |
| AI offline / error | AI chat | "Couldn't reach the assistant — try again in a moment." Retry button. |
| Image load error | Any card/hero | Grey placeholder + small mountain icon. |
| Consent pending | Global | Analytics anonymous; banner until choice. |
| Satisfaction eligible | Detail | After 2nd distinct resort view in session; prompt once. |

## Interaction Primitives

- **Tap to navigate** on cards, rows, chips, tabs, quiz answers, legal links.
- **Swipe** on onboarding cards and optional quiz (forward); standard horizontal slide, no spring overshoot.
- **Scroll** vertical on all content surfaces; horizontal scroll only on filter chip row and StatRow overflow.
- **Shared-element transition:** resort card image → detail hero, 280ms ease-out.
- **Overlay dismiss:** AI chat × top-right; onboarding skip top-right.
- **External links:** always new tab, `rel="noopener noreferrer"`, clean URLs (no affiliate params).
- **Banned Phase 1:** pull-to-refresh, swipe-to-delete, long-press menus, pinch-zoom on non-map content, hide-on-scroll chrome.

### Motion budget (from PRD §12.8)

| Interaction | Duration | Easing |
|-------------|----------|--------|
| Hero crossfade | 1s fade, 3.5s hold | linear opacity |
| Card → detail hero | 280ms | ease-out |
| Filter chip tap | 150ms scale | ease-out |
| Quiz advance | 200ms | slide from right |
| List card fade-in (first paint) | 60ms stagger | translateY 10px→0 |
| Bottom sheet (Phase 2) | 260ms | ease-out |

Max 300ms for any UI transition. No bounce physics.

## Accessibility Floor

Visual contrast targets live in `DESIGN.md`. Behavioural requirements:

- All interactive elements: accessible name + role; resort cards announce name + region.
- Tap targets ≥ 44×44px (quiz answers, chips, nav, CTA floating).
- Focus order follows visual reading order on every surface.
- `prefers-reduced-motion`: disable carousel auto-advance and list stagger; keep instant state changes.
- Unsplash attribution links: descriptive text ("Photo by [Name] on Unsplash"), not "click here".
- AI chat input: labelled; error/limit messages announced to screen readers.
- Colour is not the only indicator for filter active state — weight or underline accompanies `{colors.accent}`.

## Responsive & Platform

Phase 1 optimises **375–430px width**. Wider viewports: single-column centred column max ~430px [ASSUMPTION] or full-bleed photography with constrained text measure — implementation choice, not Phase 1 priority.

Safe-area insets respected for notched devices (bottom nav, floating CTA, chat input).

No native app shell Phase 1. Add-to-home-screen is best-effort, not designed.

## Key Flows

### Flow 1 — Alex discovers Niseko and plans arrival

**Protagonist:** Alex, intermediate rider, first Japan trip, flying CTS with partner. Overwhelmed by options; wants honest guidance.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Onboarding | Swipes 2 cards, taps "Get started" | Feels welcomed, not sold to |
| 2 | Home | Scrolls hero, taps "Deep Powder" filter | List narrows; sees Niseko card |
| 3 | Home → Detail | Taps Niseko card | **Shared-element hero transition** — "this is the real thing" |
| 4 | Detail | Reads QuickVerdict + lowlights ("crowded peak weeks") | Trust beat — honesty reduces anxiety |
| 5 | Detail | Scrolls to Getting There | Opens train option link in new tab |
| 6 | Detail | Taps "Plan with AI →" | Asks: "Is Niseko ok for my partner who's still learning?" |
| 7 | AI Chat | Gets seed-content answer citing beginner terrain | Confidence to share with partner |
| 8 | Detail (2nd resort) | Backs out, opens Hakuba for comparison | Satisfaction prompt appears once |

**Failure path:** Filter returns zero → empty state suggests broader chip; Alex taps "All" and browses full list.

**Analytics spine:** `filter_applied` → `resort_card_tapped` → `resort_detail_viewed` → `getting_there_viewed` → `external_link_clicked` → `ai_chat_opened` → `ai_chat_message_sent` → `satisfaction_prompt_submitted`.

---

### Flow 2 — Ninja validates the quiz against expert taste

**Protagonist:** Ninja, experienced snowboard instructor, many Japan seasons. Skeptical of recommendation engines; wants to see if the quiz matches his mental model.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Explore | Taps "Find my resort" CTA | Skips onboarding (returning) |
| 2 | Quiz Q1 | Selects "I charge hard" | — |
| 3 | Quiz Q2 | Selects "The powder & terrain" | — |
| 4 | Quiz Q3 | Selects "Hokkaido (Sapporo)" | — |
| 5 | Quiz Q4 | Selects "Peak powder (Feb)" | — |
| 6 | Quiz results | Sees header + **podium top 3** incl. Rusutsu / Kiroro — **#1 featured** with match reason | **Verdict moment** — is his favourite in top 3? |
| 7 | Results → Detail | Opens Rusutsu detail | Scans highlights/lowlights like a peer review |
| 8 | Detail | Lowlights mention tree-run policies or access quirks | If accurate, trust increases; if generic, trust drops |
| 9 | Detail | Optional: runs quiz again with different answers | Checks consistency — power user behaviour |

**Success for Ninja:** Quiz top results include at least one resort he respects for the stated profile; lowlights are specific, not boilerplate. He may disagree with #1 ranking but finds the logic defensible.

**Failure path:** Results feel random → Ninja filters Explore manually by Hokkaido + Deep Powder and ignores quiz future visits. Product should still deliver value via honest detail pages.

**Analytics spine:** `quiz_started` (entry: explore) → `quiz_completed` → `resort_card_tapped` (list_context: quiz) → `resort_detail_viewed` (source: card).

---

### Flow 3 — Alex takes the quiz (alternate entry)

Same as Flow 2 steps 2–7 but Alex enters from onboarding final card, selects "Getting confident" + "Family-friendly" + "Hokkaido" + "Still planning". Results skew toward approachable Hokkaido resorts. Alex taps first result → detail → Getting There. Lighter AI use or none.

---

## Mock coverage

| Surface | Mock | Notes |
|---------|------|-------|
| Home | `mockups/key-home.html` | Hero, chips, cards, nav |
| Resort Detail | `mockups/key-resort-detail.html` | Verdict, highlights/lowlights, AI CTA |
| Quiz question | `mockups/key-quiz.html` | Q2 for Ninja persona |
| Quiz results | `mockups/key-quiz-results.html` | Post-quiz list for Ninja |
| Onboarding | spine-only | Spec sufficient per PRD §12.5 Screen 0 |
| AI Chat | spine-only | Layout specified; mock deferred unless requested |
| Getting There | spine-only | Row pattern in Component Patterns |
| Info | spine-only | Minimal; no photography |

## Open questions

| Item | Status | Owner |
|------|--------|-------|
| Quiz scoring algorithm weights | Deferred to architecture | Dev |
| Quiz back-navigation | Forward-only assumed | Confirm in implementation if needed |
| Cosmetic polish pass | User noted revisit in-browser | Design during build |
| Key-screen mocks for AI Chat / Getting There | Spine-only unless requested | User |

*Spines win on conflict with any mock or import.*
