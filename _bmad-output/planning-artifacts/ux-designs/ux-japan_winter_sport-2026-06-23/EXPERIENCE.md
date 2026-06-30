---
name: Powri Phase 2 — Experience Extension
status: final
updated: 2026-06-23
extends: ../ux-japan_winter_sport-2026-06-12/EXPERIENCE.md
sources:
  - prds/prd-japan_winter_sport-2026-06-23/prd.md
  - prds/prd-japan_winter_sport-2026-06-23/addendum.md
  - epics/epics-phase2.md
  - docs/tracking-plan.md
---

# Powri Phase 2 — Experience Spine

> **Base spine:** [`../ux-japan_winter_sport-2026-06-12/EXPERIENCE.md`](../ux-japan_winter_sport-2026-06-12/EXPERIENCE.md) (Phase 1). Visual tokens: [`DESIGN.md`](DESIGN.md) + Phase 1 DESIGN.md via `{path.to.token}`. **Spines win on conflict** with mocks.

Phase 2 adds auth-gated write actions, saved resorts, map, reviews, passport, trip skeleton, and adaptive navigation. Guest browsing of editorial + read-only UGC remains unchanged.

## Foundation

**Form factor:** Mobile-first responsive web — primary 375–430px. **Desktop ≥768px** is now a first-class layout target for top nav (OQ-3 resolved). Bottom nav hidden at `md` breakpoint.

**UI system:** Tailwind + Theme A tokens. Phase 2 components per `DESIGN.md` Components. Icons: Lucide line set.

**Auth model:** Google OAuth + magic link only. Gated actions: write review, comment, report, check-in, trip CRUD, save sync across devices. Guest can browse, read reviews, save locally, open Plan empty state.

**Analytics:** Phase 2 events per PRD §10.4 — registry in `phase2Events.ts` when implementing. Respect `consent_state`.

**Phase boundary:** Live AI chat remains Phase 3 — Phase 2 shows stubs only (FR-13). Review AI theme summaries Phase 3.

## Information Architecture

| Surface | Route | Auth | Phase | Purpose |
|---------|-------|------|-------|---------|
| Home | `/` | Public | 1 | Hero + filters + resort list |
| Explore | `/explore` | Public | 1 | Discovery + quiz CTA |
| **Saved** | `/saved` | Public (guest local) | **2** | Hearted resort cards |
| **Plan** | `/plan` | Public read; CRUD login | **2** | Trip list + create |
| **Trip detail** | `/plan/[tripId]` | Owner login | **2** | Template timeline edit |
| Resort Detail | `/resorts/[slug]` | Public | 1+2 | Editorial + map + Experience |
| **Sign-in sheet** | Modal / sheet | — | **2** | Google + magic link |
| **Passport (owner)** | `/passport` | Login | **2** | Check-ins, badges, photos |
| **Public profile** | `/u/[username]` | Public read | **2** | Identity, badges, reviews excerpt |
| **Account** | `/account` | Login | **2** | Profile edit, export, delete |
| Info | `/info` | Public | 1 | About, legal, version |
| Privacy / Terms / Guidelines | `/privacy`, `/terms`, `/community` | Public | 1+2 | Legal + community rules |
| Onboarding / Quiz | `/quiz` | Public | 1 | Unchanged |
| AI Chat | Overlay | Phase 3 | 3 | **Stub modal Phase 2** |

**Resort detail section order (Phase 2):**
1. Hero + overview (SSG)
2. Highlights / lowlights (SSG)
3. Terrain / trail map link (SSG)
4. Getting There (SSG)
5. **Around the resort** — map (lazy client)
6. **How people experienced it** — reviews + comments (aggregate SSR, list hydrated)
7. Practical tips (SSG, if present)

**Navigation model (mobile <768px):**
```
Top bar: [☰] Powri / Search [♥ on detail] [Avatar/Sign in]
Bottom nav: Home · Explore · Saved · Plan (always visible)
Hamburger: Passport · Info · Account / Sign in
```

**Navigation model (desktop ≥768px):**
```
Top nav: Powri · Home · Explore · Saved · Plan · Passport · Info · [Avatar]
(no bottom nav)
```

→ Nav reference: `mockups/key-saved.html`  
→ Detail Phase 2: `mockups/key-resort-detail-phase2.html`

## Inspiration & Anti-patterns

**Embrace (Phase 2 additions)**
- Strava-style passport grid — personal, not competitive
- Airbnb saved hearts — subtle, reversible
- Google sign-in familiarity for trust
- AllTrails-style honest review cards with author identity

**Reject**
- Login walls on reading content
- Nested comment threads (Reddit/Facebook style)
- Fake urgency on sign-in ("Sign up now!")
- Leaderboards or public rank by review count
- Password fields (explicitly out of scope)
- Apple Sign-In button (deferred)

## Voice and Tone

Inherited Phase 1 rules. Phase 2 additions:

| Moment | Copy pattern |
|--------|--------------|
| Sign-in sheet | "Sign in to continue" — not "Create account" |
| Magic link sent | "Check your inbox — we sent a link to m•••@gmail.com" |
| Magic link trust | "One-time link · Expires in 15 minutes · No password" |
| Write review gate | "Sign in to share your experience" |
| Guest saved banner | "Sign in to sync saved resorts across devices" |
| Plan empty (guest) | "Sign in to start planning your trip" |
| Plan empty (user) | "No trips yet — create your first ski trip" |
| Trip quota hit | "You've reached the maximum of 5 trips" |
| Badge unlock | "You earned First Tracks" — warm, not hype |
| AI stub | "AI trip assistant coming soon — start a trip skeleton now" |
| Report confirm | "Thank you — we'll review this" |
| Profanity block | "That display name isn't allowed — try another" |

**Banned:** "JWT", "token", "OAuth" in user-facing copy.

## Component Patterns

Phase 1 components unchanged unless noted. New / updated:

| Component | Use | Behavioral rules |
|-----------|-----|------------------|
| BottomNav | Main surfaces | **4 tabs** Phase 2. Hidden ≥768px. Never hide on scroll. |
| DesktopTopNav | Desktop main surfaces | Replaces bottom nav. Same routes. Sticky. |
| AppMenuSheet | Mobile overflow | Passport, Info, Account only — **no** Home/Explore/Saved/Plan dupes. Badge count chip on Passport if ≥1 badge. |
| SaveHeart | Card + detail hero | Toggle save. Guest → localStorage. User → API. Merge on login. Fires `resort_saved` / `resort_unsaved`. |
| SignInSheet | Gated action intercept | Preserves return URL/intent. Google → OAuth redirect. Email → magic link flow. Close returns to prior surface. |
| MapSection | Detail §5 | Lazy load on viewport. Category filter chips. Pin tap → POI sheet → external Google Maps. `map_section_viewed`, `map_pin_tapped`. |
| ExperienceSection | Detail §6 | SSR aggregate rating + count. Client hydrates review list. Write review → sign-in if guest. |
| ReviewCard | Experience section | One review per user per resort. Owner can edit. Others can report (login required). Photos max 5. |
| CommentList | Experience section | Flat, newest first, max 50 + load more. No replies. |
| CheckInButton | Detail hero or passport | "I've visited" → sign-in if guest. Manual check-in; optional photo. Triggers badge evaluation. |
| BadgeUnlockToast | Global | After badge award. 4s auto-dismiss. Tap → `/passport`. |
| PassportGrid | `/passport`, public profile | Owner can add check-in. Public read-only on profile. |
| TripCard | `/plan` | Shows resort, dates, day count. Max 5 trips enforced server-side. |
| TripDayTimeline | Trip detail | Editable activity titles/notes. Template placeholders clearly labelled. |
| AIStubModal | Detail + quiz results | Replaces floating AI CTA action Phase 2. Trip CTA or sign-in. |
| SyncBanner | `/saved` guest | Dismissible per session. Sign-in CTA. |
| AccountSettingsForm | `/account` | Display name profanity filter server-side. Export JSON download. Delete → confirm → session revoked. |

**Updated Phase 1 components:**
- **CTA Floating** — label may remain "Plan with AI →" but opens **AIStubModal** not chat overlay Phase 2.
- **ResortCard** — adds SaveHeart overlay top-right.

## State Patterns

| State | Surface | Treatment |
|-------|---------|-----------|
| Guest browsing | Any public | Full editorial + read reviews; write actions → SignInSheet |
| Guest saved | `/saved` | Cards from localStorage; SyncBanner visible |
| Save merge | Post-login | Background merge; toast "Saved resorts synced" if slugs merged |
| Sign-in sheet open | Modal | Scrim tap dismisses unless submit in progress |
| Magic link pending | SignInSheet | Confirmation UI; no spinner-only state |
| Magic link landing | Return URL | Resume intended action (review form, etc.) |
| Map loading | Detail | Skeleton 240px; chips disabled until ready |
| Map error | Detail | "Map unavailable — try again later" + retry |
| Reviews loading | Experience | Skeleton cards below SSR aggregate |
| Reviews empty | Experience | "No reviews yet — be the first" + Write CTA |
| Review submitting | Review form | Disable submit; progress on photo upload |
| Comment empty | Experience | Encourage first comment; no fake seed comments |
| Check-in done | Detail / Passport | Button → "Visited [date]" disabled state |
| Badge unlock | Global | Toast + optional haptic [ASSUMPTION: no haptic web] |
| Plan empty guest | `/plan` | Illustration + sign-in CTA |
| Plan empty user | `/plan` | CTA "New trip" |
| Trip quota full | Plan create | Inline error before form submit |
| Trip template | Trip detail | Placeholder rows italic secondary colour |
| AI stub | Modal | Disabled "Fill with AI" button with tooltip "Coming in Phase 3" |
| Report submitted | Review/comment | Inline confirmation; content stays visible until admin hides |
| Account delete pending | `/account` | Confirm dialog; 30-day policy explained |
| Profile not found | `/u/[username]` | 404 friendly — "This profile doesn't exist" |
| Desktop layout | ≥768px | BottomNav unmounted; DesktopTopNav mounted |

## Interaction Primitives

Inherited Phase 1 primitives. Phase 2 additions:

- **Heart toggle** — tap save/unsave; 150ms scale; no undo toast (toggle again to undo).
- **Sign-in intercept** — any gated action opens SignInSheet with `returnTo` preserved.
- **Map chip filter** — single-select category; refreshes POI pins.
- **POI sheet** — swipe down or scrim tap dismisses.
- **Review photo add** — native file picker; client validates size/type before upload.
- **Star rating input** — tap 1–5 stars; required for review submit.
- **Report flow** — reason picker (Spam, Offensive, Off-topic, Other) → submit.
- **Trip activity edit** — inline tap row → edit title/notes → save on blur.
- **Passport check-in** — confirm dialog with optional photo attach.
- **External map link** — Google Maps opens new tab with place ID.

**Banned Phase 2:** nested comment reply UI, pull-to-refresh on UGC lists, swipe-to-delete reviews (use edit/delete from menu).

### Motion budget (Phase 2 additions)

| Interaction | Duration | Easing |
|-------------|----------|--------|
| Sign-in sheet enter | 260ms | ease-out |
| Heart fill | 150ms | ease-out |
| Badge toast enter/exit | 200ms | ease-out |
| Map section fade-in | 200ms | ease-out |
| POI bottom sheet | 260ms | ease-out |

## Accessibility Floor

Inherited Phase 1 floor. Phase 2 additions:

- SaveHeart: `aria-pressed` + "Save resort" / "Remove from saved".
- Star rating input: radio group pattern; announce selected value.
- Sign-in sheet: focus trap; first focus on Google button; Esc dismisses.
- Map: list alternative for POI results below map [ASSUMPTION: optional screen-reader list].
- Review photos: alt text from user caption or "Review photo N".
- Badge tiles: locked state announced as "Locked badge: {name}".
- Report dialog: labelled reason radio group.

## Responsive & Platform

| Breakpoint | Behaviour |
|------------|-----------|
| 375–767px | 4-item bottom nav; hamburger overflow; sign-in bottom sheet |
| ≥768px | Top nav only; sign-in centred modal max 400px; badge grid 4–5 cols |
| ≥1024px | Plan/Account content max-width 720px centred |

Safe-area insets: bottom nav, floating CTAs, sign-in sheet, toast.

No native app Phase 2. PWA add-to-home-screen best-effort.

## Key Flows

### Flow 4 — Yuki maps Niseko and saves for later (UJ-1)

**Protagonist:** Yuki, intermediate snowboarder, first Japan trip, planning from Singapore.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Explore | Opens Niseko United detail | Editorial trust unchanged from Phase 1 |
| 2 | Detail | Scrolls to **Around the resort** | Map lazy-loads; village context appears |
| 3 | Map | Taps restaurant pin | **POI sheet** — understands dining options nearby |
| 4 | Map | Opens Google Maps externally | Confirms real place without leaving Powri permanently |
| 5 | Detail | Taps **heart** on hero | Saved — guest localStorage |
| 6 | Saved | Opens Saved tab | Niseko card present; sync banner shown |

**Analytics:** `resort_detail_viewed` → `map_section_viewed` → `map_pin_tapped` → `resort_saved` (`is_logged_in: false`).

---

### Flow 5 — Marcus reviews Hakuba and earns First Tracks (UJ-2)

**Protagonist:** Marcus, returning visitor, just finished Hakuba trip.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Detail | Scrolls to Experience; taps **Write a review** | SignInSheet — Google sign-in |
| 2 | SignInSheet | Continues with Google | Returns to review form |
| 3 | Review form | 5 stars + text + 1 photo | Submit |
| 4 | Experience | Review appears with name + avatar | **Public accountability** beat |
| 5 | Global | **First Tracks** badge toast | Reward without leaderboard |
| 6 | Passport | Views check-in grid | Optional plan return trip CTA → UJ-4 |

**Analytics:** `auth_sign_in_completed` → `review_submitted` → `badge_unlocked`.

---

### Flow 6 — Lin collects Hokkaido Explorer (UJ-3)

**Protagonist:** Lin, season pass holder, visiting multiple Hokkaido resorts.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Detail (Furano) | **I've visited** check-in | Logged in — no friction |
| 2 | Detail (Rusutsu) | Second check-in | Passport grid grows |
| 3 | Detail (Kiroro) | Third Hokkaido check-in | **Hokkaido Explorer** toast |
| 4 | Passport | Reviews badge shelf | Shares `/u/lin` URL with friends |

---

### Flow 7 — Priya creates February trip skeleton (UJ-4)

**Protagonist:** Priya, organised planner, dates locked, wants structure before AI.

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Plan | Taps **New trip** | Create form |
| 2 | Create form | Selects Niseko, Feb 10–15 | Template generates |
| 3 | Trip detail | Sees Day 1 transfer → Day 2–4 ski placeholders | **Climax:** multi-day structure exists |
| 4 | Trip detail | Edits Day 3 note "Try Grand Hirafu" | Personalisation without AI |
| 5 | Trip detail | Sees disabled **Fill with AI** | Phase 3 tease |

---

### Flow 8 — Marcus reports offensive review (UJ-5)

| Step | Surface | Action | Climax / beat |
|------|---------|--------|---------------|
| 1 | Experience | Finds offensive review | — |
| 2 | ReviewCard | Taps Report → reason | Submit (login required) |
| 3 | Inline | "Thank you — we'll review this" | Trust in moderation |
| 4 | (Admin) | Hides via API | Content removed on refresh |

**Guest edge:** Report tap → SignInSheet first.

---

## Mock coverage

| Surface | Mock | Notes |
|---------|------|-------|
| Sign-in sheet | `mockups/key-sign-in-sheet.html` | Google primary, magic link secondary |
| Saved list | `mockups/key-saved.html` | 4-tab nav + guest banner |
| Plan / trips | `mockups/key-plan-trips.html` | Trip cards + empty state |
| Resort detail Phase 2 | `mockups/key-resort-detail-phase2.html` | Map + Experience sections |
| Passport | `mockups/key-passport.html` | Badge grid + check-ins |
| Public profile | `mockups/key-public-profile.html` | `/u/username` |
| Trip detail | spine-only | Timeline spec in Component Patterns |
| Account settings | spine-only | Form pattern in Component Patterns |
| Review form | spine-only | Opens from Experience CTA |
| Desktop top nav | spine-only | Described in IA; implement from tokens |

## Open questions

| Item | Status |
|------|--------|
| Review form as full page vs sheet | Sheet on mobile, dedicated route on desktop [ASSUMPTION] |
| Check-in button placement | Detail hero overflow menu + passport page both [ASSUMPTION] |

*Spines win on conflict with any mock or import.*
