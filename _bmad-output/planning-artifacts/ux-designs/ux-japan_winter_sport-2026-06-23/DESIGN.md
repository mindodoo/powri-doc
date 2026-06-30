---
name: Powri Phase 2 — Visual Extension
description: Phase 2 component and token extensions atop Theme A. Base visual identity unchanged.
status: final
updated: 2026-06-23
extends: ../ux-japan_winter_sport-2026-06-12/DESIGN.md
sources:
  - prds/prd-japan_winter_sport-2026-06-23/prd.md
  - prds/prd-japan_winter_sport-2026-06-23/addendum.md
  - epics/epics-phase2.md
colors:
  badge-bronze: '#B87333'
  badge-silver: '#9CA3AF'
  badge-gold: '#C9A227'
  badge-locked: 'rgba(110, 106, 99, 0.4)'
  map-pin-resort: '#2D6BE4'
  map-pin-poi: '#6E6A63'
  star-filled: '#C4873A'
  star-empty: '#E8E5E0'
  sign-in-scrim: 'rgba(0,0,0,0.45)'
  toast-bg: '#1C1A17'
  toast-text: '#FFFFFF'
  sync-banner-bg: '#EEF3FF'
typography:
  review-author:
    fontFamily: "'DM Sans', system-ui, sans-serif"
    fontSize: 14px
    fontWeight: '600'
    lineHeight: '1.3'
  trip-day-label:
    fontFamily: "'Cormorant Garamond', Georgia, serif"
    fontSize: 20px
    fontWeight: '500'
    lineHeight: '1.2'
rounded:
  sheet: 16px
  avatar: 99px
  map: 12px
  toast: 10px
spacing:
  bottom-nav-items: 4
  badge-grid-gap: 16px
  review-photo-gap: 8px
components:
  bottom-nav:
    height: 56px
    itemCount: 4
    labels: 'Home · Explore · Saved · Plan'
  desktop-top-nav:
    height: 56px
    breakpoint: 768px
  save-heart:
    size: 44px
    activeColor: '{colors.accent}'
    inactiveColor: '{colors.text-secondary}'
  sign-in-sheet:
    maxWidth: 400px
    borderRadius: '{rounded.sheet}'
  map-section:
    height: 240px
    borderRadius: '{rounded.map}'
  review-card:
    borderRadius: '{rounded.card}'
    padding: '{spacing.card-padding}'
  badge-tile:
    size: 96px
    borderRadius: 99px
  badge-toast:
    duration: 4s
    borderRadius: '{rounded.toast}'
  trip-card:
    borderRadius: '{rounded.card}'
    minHeight: 88px
---

# Powri Phase 2 — Design Extension

> **Base spine:** [`../ux-japan_winter_sport-2026-06-12/DESIGN.md`](../ux-japan_winter_sport-2026-06-12/DESIGN.md) (Theme A). All `{colors.*}`, `{typography.*}`, `{rounded.*}`, and `{spacing.*}` from Phase 1 apply unless overridden here. **Spines win on conflict** with mocks.

Phase 2 adds auth, UGC, map, passport, and trip surfaces without changing the editorial magazine aesthetic. New UI stays quiet — photography and typography still lead.

→ Phase 2 mocks: `mockups/key-sign-in-sheet.html`, `key-saved.html`, `key-plan-trips.html`, `key-resort-detail-phase2.html`, `key-passport.html`, `key-public-profile.html`

## Brand & Style

Phase 2 introduces **community identity** (avatars, badges, reviews) inside the same warm editorial frame. Badges are celebratory but restrained — tier rings, not casino graphics. UGC cards use `{colors.surface}` on `{colors.bg}` with `{colors.divider}` separators, matching ResortCard hierarchy.

Auth moments should feel **trustworthy, not salesy** — Google button first; magic link framed as convenience, not novelty.

Reject: leaderboard rankings, streak counters, notification badges on every tab, red report buttons, star-rating widgets that look like e-commerce.

## Colors

**Inherited:** all Phase 1 tokens unchanged.

**Phase 2 additions:**

| Token | Value | Usage |
|-------|-------|-------|
| `{colors.badge-bronze}` | `#B87333` | Bronze tier badge ring |
| `{colors.badge-silver}` | `#9CA3AF` | Silver tier badge ring |
| `{colors.badge-gold}` | `#C9A227` | Gold tier badge ring |
| `{colors.badge-locked}` | 40% grey | Locked badge tile overlay |
| `{colors.map-pin-resort}` | `{colors.accent}` | Primary resort pin |
| `{colors.map-pin-poi}` | `{colors.text-secondary}` | Google Places POI pins |
| `{colors.star-filled}` | `{colors.accent-warm}` | Review star rating (warm, not red) |
| `{colors.star-empty}` | `{colors.divider}` | Unfilled stars |
| `{colors.sign-in-scrim}` | `rgba(0,0,0,0.45)` | Sign-in sheet backdrop |
| `{colors.toast-bg}` | `{colors.text-primary}` | Badge unlock toast |
| `{colors.sync-banner-bg}` | `{colors.highlight-bg}` | Guest saved-list sync banner |

## Typography

**Inherited:** display, body, caption, badge, nav-label from Phase 1.

| Token | Usage |
|-------|-------|
| `{typography.review-author}` | Review card author name (links to profile) |
| `{typography.trip-day-label}` | Trip detail day headers ("Day 2 — Ski") |

Review body text uses `{typography.body}`. Comment text uses `{typography.body}` at 14px. Aggregate rating number uses `{typography.section-heading}`.

## Layout & Spacing

**Mobile primary:** 375–430px unchanged.

**Desktop (≥768px):** hide `{components.bottom-nav}`; show `{components.desktop-top-nav}`. Content max-width 720px centred for Plan, Account, Passport owner view. Resort detail remains full-bleed hero with constrained text measure below fold.

**Badge grid:** 3 columns mobile, 4–5 desktop; `{spacing.badge-grid-gap}`.

**Experience section:** full `{spacing.screen-x}` margins; map `{components.map-section.height}` before lazy load placeholder.

## Elevation & Depth

**Sign-in sheet:** `{colors.sign-in-scrim}` scrim; sheet `{colors.surface}` with top `{rounded.sheet}` corners on mobile bottom sheet.

**Map POI bottom sheet:** same pattern as Phase 1 map bottom sheet spec — `{colors.overlay-scrim}` + `{colors.surface}` sheet, 260ms ease-out.

**Review photos:** no shadow; 8px gap horizontal scroll strip.

**Badge unlock toast:** `{colors.toast-bg}` pill above bottom nav / bottom safe area; subtle `0 4px 12px rgba(0,0,0,0.15)`.

## Shapes

| Token | Applied to |
|-------|------------|
| `{rounded.sheet}` | Sign-in sheet, map POI sheet top corners |
| `{rounded.avatar}` | Profile avatars, review author avatars |
| `{rounded.map}` | Map container inset on detail page |
| `{rounded.toast}` | Badge unlock toast |

## Components

### BottomNav (Phase 2)

Four tabs: **Home · Explore · Saved · Plan**. Same `{components.bottom-nav.height}`. Active: `{colors.accent}` label. Icons: Lucide — `Home`, `Compass`, `Heart`, `CalendarDays`. Hidden at `md` (768px)+.

→ Reference: `mockups/key-saved.html`, `mockups/key-plan-trips.html`

### DesktopTopNav

Horizontal links: Home · Explore · Saved · Plan · Passport · Info · Sign in / Avatar. Sticky `{components.desktop-top-nav.height}`; `{colors.surface}` + `{colors.divider}` bottom border. No bottom nav on desktop.

### AppMenuSheet (overflow)

Mobile hamburger only. Items: **Passport** (badge count chip if ≥1), **Info**, **Account** or **Sign in**. Does **not** duplicate bottom-nav destinations.

### SaveHeart

44×44px tap target on resort card (top-right overlay) and detail hero (top-right, below safe area). Outline heart inactive; filled `{colors.accent}` active. 150ms scale animation on toggle — no particle burst.

### SignInSheet

Bottom sheet (mobile) / centred modal (desktop). Order:
1. Title: "Sign in to continue"
2. **Continue with Google** — full-width primary `{colors.accent}` button
3. Divider: "or"
4. Email field + **Email me a secure link** — secondary outline button
5. Trust line: 🔒 "Link expires in 15 minutes · Works once"
6. Privacy link: `{typography.caption}`

Post-submit: confirmation state with masked email + "Open Gmail" deep link where supported.

→ Reference: `mockups/key-sign-in-sheet.html`

### MapSection

Lazy-loaded `{components.map-section.height}` container with `{rounded.map}`. Resort pin centred; category chips above map (Restaurants · Onsen · Convenience). "Powered by Google" attribution `{typography.caption}` required. Skeleton: grey box + map icon until intersection observer fires.

POI tap → bottom sheet with name, rating, vicinity, "Open in Google Maps" external link.

→ Reference: `mockups/key-resort-detail-phase2.html`

### ExperienceSection

Section header `{typography.badge}`: "HOW PEOPLE EXPERIENCED IT". Aggregate ★ + count in `{typography.section-heading}`. **Write a review** — `{rounded.button}` outline CTA.

### ReviewCard

`{colors.surface}` card, `{rounded.card}`. Row: `{rounded.avatar}` 40px + `{typography.review-author}` + date `{typography.caption}`. Star row `{colors.star-filled}`. Body `{typography.body}`. Photo strip horizontal scroll. Kebab menu: Edit (owner) · Report (others, requires login).

### CommentRow

Flat list — no indent, no thread lines. Avatar 32px + name + timestamp. Body max 500 chars displayed. **Add a comment** inline at list bottom.

### BadgeTile

`{components.badge-tile.size}` circle. Tier ring 2px `{colors.badge-bronze|silver|gold}`. Locked: `{colors.badge-locked}` + dashed ring. Name below in `{typography.caption}` — never inside icon.

### BadgeUnlockToast

Non-blocking; shows badge icon + name + "View passport →". Auto-dismiss `{components.badge-toast.duration}`.

### PassportGrid

Resort visit tiles — circular resort thumbnail or initial; check-in date `{typography.caption}`. Empty slots show "+" for manual check-in CTA on owner view.

→ Reference: `mockups/key-passport.html`

### PublicProfileHeader

Large `{rounded.avatar}` 80px, display name `{typography.section-heading}`, bio `{typography.body}`, badge shelf horizontal scroll, passport grid excerpt, review list excerpts.

→ Reference: `mockups/key-public-profile.html`

### TripCard

`{components.trip-card}` — resort name `{typography.card-title}`, date range `{typography.caption}`, progress chip ("3 days · Template"). Chevron right. Tap → trip detail.

### TripDayTimeline

Vertical timeline: `{typography.trip-day-label}` per day. Activity rows — time optional, title `{typography.body}`, notes `{typography.caption}`. Placeholder activities styled with `{colors.text-secondary}` italic hint "(template — edit me)".

### SyncBanner

Guest-only on `/saved`. `{colors.sync-banner-bg}` full-width banner: "Sign in to sync saved resorts across devices" + dismiss ×.

### AIStubModal

Replaces live AI chat Phase 2. Copy: "AI trip assistant coming soon." Primary CTA: **Start a trip skeleton** (logged in) or **Sign in to plan** (guest). Secondary: Dismiss.

### AccountSettings

Form rows: display name, bio, avatar upload. Destructive section separated by `{spacing.section-gap}`: Export my data · Delete account · Sign out.

## Do's and Don'ts

**Do**
- Keep Google sign-in visually primary over magic link
- Show aggregate rating in SSR HTML for SEO
- Use warm amber stars, not red/yellow e-commerce stars
- Credit Google Places on map
- Let badge unlocks feel like a quiet reward, not a popup ad

**Don't**
- Nest comments or show Reddit-style threads
- Gate reading reviews behind login
- Show fake POI data in trip templates
- Use gamification streaks or daily login rewards
- Index public profiles in Phase 2 (`noindex`)

*Spines win on conflict with any mock or import.*
