---
name: Japan Ski Trip Planner
description: Mobile-first editorial travel companion for Japan ski trip planning. Premium magazine aesthetic — photography-forward, calm, trustworthy.
status: final
updated: 2026-06-12
sources:
  - japan_ski_prd.md#12-uiux-specification
  - _bmad-output/planning-artifacts/epics.md
colors:
  bg: '#F7F5F2'
  surface: '#FFFFFF'
  text-primary: '#1C1A17'
  text-secondary: '#6E6A63'
  accent: '#2D6BE4'
  accent-warm: '#C4873A'
  divider: '#E8E5E0'
  highlight-bg: '#EEF3FF'
  lowlight-bg: '#FFF4E3'
  nav-bg: '#FFFFFF'
  on-accent: '#FFFFFF'
  overlay-scrim: 'rgba(0,0,0,0.3)'
  hero-gradient: 'linear-gradient(to top, rgba(0,0,0,0.55) 0%, transparent 60%)'
typography:
  display:
    fontFamily: "'Cormorant Garamond', Georgia, serif"
    fontSize: 40px
    fontWeight: '600'
    lineHeight: '1.15'
  display-italic:
    fontFamily: "'Cormorant Garamond', Georgia, serif"
    fontSize: 24px
    fontWeight: '400'
    lineHeight: '1.25'
    fontStyle: italic
  section-heading:
    fontFamily: "'Cormorant Garamond', Georgia, serif"
    fontSize: 26px
    fontWeight: '500'
    lineHeight: '1.2'
  card-title:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 18px
    fontWeight: '600'
    lineHeight: '1.3'
  body:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 15px
    fontWeight: '400'
    lineHeight: '1.6'
  caption:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 12px
    fontWeight: '400'
    lineHeight: '1.4'
  badge:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 11px
    fontWeight: '500'
    lineHeight: '1.2'
    letterSpacing: 0.08em
  nav-label:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 11px
    fontWeight: '500'
    lineHeight: '1.2'
rounded:
  card: 14px
  button: 10px
  badge: 99px
  pill: 99px
  image: 0px
spacing:
  '1': 8px
  '2': 16px
  '3': 24px
  '4': 32px
  '5': 48px
  screen-x: 20px
  card-gap: 20px
  section-gap: 48px
  card-padding: 16px
components:
  bottom-nav:
    height: 56px
    background: '{colors.nav-bg}'
    borderTop: '1px solid {colors.divider}'
  resort-card:
    borderRadius: '{rounded.card}'
    imageAspect: '16/10'
  filter-chip-active:
    background: '{colors.accent}'
    color: '{colors.on-accent}'
  filter-chip-inactive:
    background: '{colors.surface}'
    border: '1px solid {colors.divider}'
  cta-floating:
    height: 52px
    background: '{colors.accent}'
    color: '{colors.on-accent}'
    borderRadius: '{rounded.button}'
  chat-bubble-user:
    background: '{colors.accent}'
    color: '{colors.on-accent}'
  chat-bubble-ai:
    background: '{colors.surface}'
    border: '1px solid {colors.divider}'
---

## Brand & Style

Japan Ski Trip Planner should feel like a **trusted travel magazine that also works** — not a booking engine, not a resort brochure. Photography leads; UI chrome stays quiet. Warm off-white surfaces (`{colors.bg}`) replace clinical pure white. Serif display type carries editorial confidence; sans-serif handles everything functional.

The reference hierarchy when sources conflict: **Kinfolk** (mood and breathing room) → **Hokuoh Kurashii** (mobile structure) → **Monocle** (information rigour) → **Airbnb** (filter/search/nav interaction patterns only).

Honesty is part of the aesthetic: lowlights get amber warmth, not alarm red. The product earns trust by showing what to watch out for, not by overselling powder days.

→ Visual anchors: `mockups/key-home.html`, `mockups/key-resort-detail.html`

## Colors

- **`{colors.bg}` (`#F7F5F2`)** — page canvas. Always slightly warm; never `#FFFFFF` as a page background.
- **`{colors.surface}` (`#FFFFFF`)** — cards, chat AI bubbles, modals, sticky header when scrolled.
- **`{colors.text-primary}` (`#1C1A17`)** — headings and body on light surfaces.
- **`{colors.text-secondary}` (`#6E6A63`)** — labels, captions, attribution, secondary metadata.
- **`{colors.accent}` (`#2D6BE4`)** — primary CTAs, active filter chips, highlight badge labels, user chat bubbles, nav active indicator. The single chromatic action colour.
- **`{colors.accent-warm}` (`#C4873A`)** — lowlight badges and warm secondary accents only. Never used for primary actions.
- **`{colors.highlight-bg}` / `{colors.lowlight-bg}`** — tinted row backgrounds in the highlights/lowlights grid. Subtle, not saturated.
- **`{colors.divider}` (`#E8E5E0`)** — section rules, card borders, inactive chip outlines.

Avoid: gradients on UI chrome (hero image gradient overlay is the exception), saturated rainbow badges, pure-black text, red error fills on editorial content, decorative colour outside accent + warm accent roles.

**Future themes:** Themes B (Hokuoh), C (Monocle), D (Airbnb) reuse the same token *names* with different values via `data-theme` on `<html>`. Phase 1 ships Theme A only.

## Typography

Load from Google Fonts: `Cormorant+Garamond:ital,wght@0,500;0,600;1,400` and `DM+Sans:wght@400;500;600`.

| Role | Token | Usage |
|------|-------|-------|
| Hero titles, resort names | `{typography.display}` | Home tagline overlay, detail hero name |
| Pull quotes, quick verdict | `{typography.display-italic}` | Overlaid on hero with `{colors.hero-gradient}` backing |
| Section titles | `{typography.section-heading}` | "Highlights", terrain section |
| Card titles | `{typography.card-title}` | Resort card name, transport method |
| Body copy | `{typography.body}` | Descriptions, tips, chat AI text |
| Captions, attribution | `{typography.caption}` | Unsplash credit, season labels, dates |
| Badges, section labels | `{typography.badge}` | Uppercase, `{typography.badge.letterSpacing}` |
| Bottom nav | `{typography.nav-label}` | Tab labels |

Display type is rare and intentional. Most UI is `{typography.body}` or `{typography.caption}`. No all-caps display headings.

## Layout & Spacing

Base grid: **8px**. All spacing values are multiples of 8.

| Token | Value | Usage |
|-------|-------|-------|
| `{spacing.screen-x}` | 20px | Horizontal page margins |
| `{spacing.card-gap}` | 20px | Gap between resort cards |
| `{spacing.section-gap}` | 48px | Between major detail sections |
| `{spacing.card-padding}` | 16px | Inner card padding |

**Viewport:** 375–430px width primary. Single-column always in Phase 1. Hero carousel height ~65vh on Home; detail hero ~55vh.

**Home top bar:** transparent over hero; transitions to `{colors.surface}` + `1px {colors.divider}` border after scroll past hero.

**Bottom nav:** always visible, `{components.bottom-nav.height}` — never hide on scroll.

## Elevation & Depth

Default and Hokuoh themes avoid heavy shadows. Hierarchy comes from layout, typography, and photography — not elevation.

- **Cards:** distinguished by `{colors.surface}` on `{colors.bg}` or `{spacing.card-gap}` whitespace; optional `0 2px 10px rgba(0,0,0,0.05)` on floating elements only.
- **Sticky header (scrolled):** flat white + hairline border — no shadow.
- **AI chat overlay:** full-screen; backdrop not required (sheet covers viewport).
- **Map bottom sheet (Phase 2):** `{colors.overlay-scrim}` backdrop; sheet on `{colors.surface}`.

No hard shadows on editorial content blocks. Borders over shadows.

## Shapes

| Token | Value | Applied to |
|-------|-------|------------|
| `{rounded.card}` | 14px | Resort cards, content cards, AI bubbles |
| `{rounded.button}` | 10px | CTAs, quiz answer buttons, link chips |
| `{rounded.badge}` | 99px | Filter chips, stat pills, region tags |
| `{rounded.image}` | 0px | Hero and card photos are full-bleed or edge-to-edge within card radius |

Imagery clips to container corners. No circular crop on resort photography.

## Components

### BottomNav
Three tabs Phase 1: Home · Explore · Info. Active tab: `{colors.accent}` label + underline or dot. Icons: Lucide line icons, 24px. Spec: `{components.bottom-nav}`.

### HeroCarousel
Full-bleed, auto-advance crossfade only (no slide). 3.5s hold, 1s fade. White gradient at bottom for tagline legibility. No visible controls or dots on Home hero. Unsplash attribution overlaid or immediately below.

### ResortCard
Full-width `{components.resort-card.imageAspect}` image, `{typography.card-title}` name, region pill, 2–3 `BadgePill`s, chevron. Tap target = entire card. Image is the dominant element.

### FilterChip
Single-select horizontal scroll row. Active: `{components.filter-chip-active}`. Inactive: `{components.filter-chip-inactive}`. Tap scale 0.97→1.0, 150ms — no bounce.

### QuickVerdict
`{typography.display-italic}` white text on hero lower third. Thin white rule separates from resort name. `{colors.hero-gradient}` ensures legibility.

### HighlightLowlightRow
50/50 columns. Left header "Highlights" in `{colors.accent}`; right "Watch out for" in `{colors.accent-warm}`. Rows on `{colors.highlight-bg}` / `{colors.lowlight-bg}`.

### StatRow
Horizontal scroll or wrap of stat chips — runs, vertical drop, season length, avg snowfall. Neutral grey or highlight tint pills.

### BadgePill
Variants: highlight (blue tint + `{colors.accent}` label), lowlight (amber tint), neutral (grey). Icon 16px + `{typography.badge}` label.

### TransportRow
Icon + method + duration + cost range + external link chip. Full-width tappable row separated by `{colors.divider}`.

### OnboardingCard
Full-screen, `{colors.bg}` background, one 80×80px line icon max, no photography. Progress dots + skip top-right.

### ResortQuizCard
Full-screen question in `{typography.section-heading}`. Answer buttons full-width, `{rounded.button}`, min-height 52px for tap target.

### ChatBubble
User: `{components.chat-bubble-user}`, right-aligned. AI: `{components.chat-bubble-ai}`, left-aligned. AI may embed mini resort/transport cards inline.

### UnsplashAttribution
`{typography.caption}` on `{colors.text-secondary}`. Copy: *Photo by [Name] on Unsplash*. Links include `utm_source=japan_ski_planner&utm_medium=referral`. Visible on every image — not buried.

### SectionHeader
Uppercase `{typography.badge}` label + optional "See all →" in `{colors.accent}`. Top rule: `1px solid {colors.divider}`.

### CTA Floating ("Plan with AI →")
Pinned above BottomNav. Spec: `{components.cta-floating}`. Full width minus `{spacing.screen-x}` margins.

## Do's and Don'ts

**Do**
- Let photography dominate Home and detail heroes
- Show season labels on all T2 pricing/date fields
- Credit Unsplash on every image
- Use honest lowlights copy styling — amber, not red
- Keep motion under 300ms; crossfade and ease-out only

**Don't**
- Use hero video loops (static imagery only, Phase 1)
- Hide bottom navigation on scroll
- Use bounce/spring physics or particle effects
- Replace photography with large decorative illustrations
- Use affiliate styling or "Book now" urgency patterns
- Show streak counters, gamification, or promotional exclamation marks

*Spines win on conflict with any mock or import.*
