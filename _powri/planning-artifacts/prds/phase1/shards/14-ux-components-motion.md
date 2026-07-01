<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §12.6–12.11: Components, Motion & A11y

**Source:** [`../prd-phase1.md`](../prd-phase1.md) §12.6–12.11 · **Visual spec:** [`ux-designs/ux-phase1/design.md`](../../../ux-designs/ux-phase1/design.md)

---

## §12.6 Component library (Phase 1)

| Component | Purpose |
|-----------|---------|
| `ResortCard` | Full-width image, name, region, badges, chevron |
| `BadgePill` | Icon + label; highlight/lowlight/neutral variants |
| `HighlightLowlightRow` | 50/50 highlights / lowlights |
| `QuickVerdict` | Italic serif quote on hero gradient |
| `StatRow` | 3–4 stat chips |
| `TransportRow` | Icon, method, duration, cost, link |
| `FilterChip` | Single-select, instant filter |
| `OnboardingCard` | Full-screen swipeable intro |
| `ResortQuizCard` | Full-screen quiz question + wide answers; Serious/Cosmic copy |
| `QuizResultsReveal` | Quiz results header; Cosmic interstitial |
| `QuizTopMatchCard` | Quiz #1 — featured card, match reason, accent border |
| `QuizRunnerUpRow` | Quiz #2/#3 — minimal row with 48px thumb |
| `ChatBubble` | User (right) / AI (left) |
| `SectionHeader` | Uppercase label + optional "See all →" |
| `HeroCarousel` | Crossfade, auto-advance, no controls |
| `BottomNav` | Home · Explore · Info |
| `UnsplashAttribution` | Required on every image |

Phase 2+: `AccommodationCard`, `RestaurantCard`, `MapPin`, etc.

---

## §12.8 Motion

**Principle:** Calm, purposeful — no bounce, no particles.

| Interaction | Duration | Style |
|-------------|----------|-------|
| Hero crossfade | 1s | opacity only |
| Card → detail | 280ms | shared element, ease-out |
| Filter chip tap | 150ms | scale 0.97→1 |
| Quiz advance | 200ms | slide from right |
| Search expand | 200ms | slide down |

---

## §12.11 Accessibility

- Colour contrast on design tokens
- Tap targets ≥ **44px** (quiz answers, chips, nav)
- Semantic HTML for nav and sections
- Keyboard focus visible

---

## §12.10 Empty & loading states

Skeleton cards for resort list; friendly empty states for zero filter results (link to clear filters / try quiz).

**Full tables:** [`../prd-phase1.md`](../prd-phase1.md) §12.6–12.11
