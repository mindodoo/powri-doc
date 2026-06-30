---
title: Powri Phase 2 — Community, Maps & Trip Foundation
created: 2026-06-23
updated: 2026-06-23
revision: 3
status: final
phase: 2
builds_on: Phase 1 (launched 2026-06-17)
precedes: Phase 3 (AI Assistant, Admin UI, Review Summaries)
---

# PRD: Powri Phase 2 — Community, Maps & Trip Foundation

## 0. Document Purpose

This PRD defines Phase 2 of **Powri** — a mobile-first web product helping international and domestic skiers discover and plan Japan snow trips. Phase 1 (launched) ships resort discovery, detail pages, Getting There, and the Find My Resort quiz. Phase 2 adds **user accounts**, **community features**, **resort maps**, **Ski Passport gamification**, and a **trip-planning skeleton** that Phase 3 AI will populate.

**Audience:** Product, engineering, UX, analytics.  
| **Builds on:** [`prd/01-overview-and-goals.md`](../../prd/01-overview-and-goals.md), [`prd/02-phase1-functional-requirements.md`](../../prd/02-phase1-functional-requirements.md), [`architecture.md`](../../architecture.md).  
**Phase 2 implementation:** [`architecture-phase2.md`](../../architecture-phase2.md).  
**Technical depth:** [`addendum.md`](addendum.md) (storage, auth setup, badge UI spec, AI handoff schema).  
**Pre-implementation checklist:** [`readiness-checklist.md`](readiness-checklist.md).  
**Decision audit trail:** [`.decision-log.md`](.decision-log.md).

---

## 1. Vision

Powri Phase 2 transforms the product from a read-only editorial guide into a **warm, community-driven planning companion**. Users can explore resorts freely (no login wall), then sign in when they want to share experiences, collect Ski Passport badges, or start building a trip.

Trip planning in Phase 2 is intentionally a **template skeleton** — not a blank form for power users. Phase 3 AI will fill the skeleton from seed content and user context; users edit the AI output. This positions Powri as an assistant that saves research time for the majority, rather than competing with spreadsheets for super-planners.

The quiz remains a **standalone fun entry point** (Phase 1 behaviour). Quiz results are not stored on user profiles in Phase 2. AI assistance (Phase 3) gets **entry-point stubs** on quiz results and resort detail pages in Phase 2, but no live chat.

**Design continuity:** Premium travel-magazine aesthetic (Theme A — warm off-white, Cormorant Garamond + DM Sans). New surfaces (Passport, Trip, Reviews) must feel like the same product, not a bolt-on social network.

---

## 2. Target User

### 2.1 Jobs To Be Done

- **Discover the right resort** — browse, filter, quiz (Phase 1; unchanged).
- **See what's around a resort** — on-mountain context plus nearby food, onsen, attractions (Phase 2 map).
- **Trust peer experiences** — read and write public reviews tied to a visible profile (Phase 2).
- **Celebrate visits** — manual check-in, optional photos, earn playful badges (Phase 2 Ski Passport).
- **Start a trip plan** — create a structured itinerary shell that AI will enrich later (Phase 2 skeleton → Phase 3 AI fill).
- **Return without friction** — persistent session on same device; login only when needed.

### 2.2 Non-Users (Phase 2)

- Users who want live booking, price comparison, or real-time inventory (out of scope).
- Users expecting AI trip generation in Phase 2 (deferred to Phase 3).
- Admin moderation workflows via dedicated admin UI (deferred to Phase 3; API-level moderation in Phase 2).

### 2.3 Key User Journeys

**UJ-1. Yuki discovers Niseko, then checks what's nearby.**  
Yuki (intermediate snowboarder, first Japan trip) browses Explore, opens Niseko United detail. She scrolls editorial content (public, no login). She taps **Map** — sees resort pin plus Google Places categories (restaurants, onsen, convenience). She taps a restaurant pin; external Google Maps opens. **Climax:** she understands the village without leaving Powri's editorial frame. **Resolution:** she taps the **heart** to save Niseko to her Saved list (stored on device as a guest); she can sign in later to sync across devices.

**UJ-2. Marcus shares his Hakuba experience after login.**  
Marcus finished a trip. On Hakuba Happo detail he taps **Write a review**. A sign-in sheet appears (Google or email — not required to browse). After auth, he submits a star rating, short text, and one photo. His display name and avatar appear on the review. **Climax:** his review is live publicly; he sees a **First Tracks** badge unlock on his Ski Passport. **Resolution:** he views his profile passport grid; optional "Plan a return trip" CTA leads to trip skeleton (UJ-4).

**UJ-3. Lin collects badges across a season.**  
Lin visits three Hokkaido resorts over winter. On each detail page she taps **I've visited** (manual check-in). She optionally uploads a lift-pass photo to her passport entry (not required). After her third Hokkaido resort she earns **Hokkaido Explorer**. After five total resorts, **Powder Hunter** unlocks. **Climax:** badge toast + passport grid update. **Resolution:** she shares her passport URL (public read) or screenshot; Slope account linking remains future/optional.

**UJ-4. Priya starts a trip skeleton for February.**  
Priya is logged in. From resort detail or nav she opens **Plan** → **New trip**. She picks one primary resort, dates, and sees a **template itinerary**: Day 1 airport → transfer → check-in; Day 2 ski; Day 3 ski + dinner placeholder; etc. Activities are editable titles/notes only — no real hotel/restaurant data yet. She saves; trip count is 3 of 5 max. **Climax:** structured multi-day view exists. **Resolution:** Phase 3 "Fill with AI" entry point visible but disabled/teased until AI ships.

**UJ-5. User reports inappropriate content.**  
Marcus (logged in) reads a review with offensive language on a resort detail page. He taps **Report** on the review, chooses a reason, and submits. The report is queued for admin review. **Climax:** he sees a confirmation — "Thank you, we'll review this." **Resolution:** an admin hides the review via API; it no longer renders. **Edge case:** guests who tap Report are prompted to sign in first — reporting requires authentication to prevent spam.

---

## 3. Glossary

| Term | Definition |
|------|------------|
| **Powri** | Product brand (Japan ski/snowboard trip planner web app). |
| **Resort** | One of 20 seed resorts; content from `content/resorts/<slug>.md`; always public. |
| **Guest** | Unauthenticated visitor; browse, quiz, view public reviews/comments, save resorts locally (device only). |
| **User** | Authenticated account; review, comment, trips, passport, synced saves, report content. |
| **Admin** | User with `role = admin` on profile; hide/delete UGC via API/DB (UI Phase 3). |
| **Public profile** | Read-only page at `/u/:username` showing display name, badges, passport grid, and public reviews. |
| **Ski Passport** | User's gamified visit log + badge collection (Hall of Fame). |
| **Check-in** | Manual "I've visited this resort" action; no evidence required. |
| **Trip** | User-owned itinerary object (max 5 per user); template activities in Phase 2. |
| **Activity** | Single item in a trip day (airport, transport, accommodation, ski, restaurant, attraction, custom). |
| **Template itinerary** | Pre-structured day/activity placeholders — not live POI/booking data. |
| **Seed content** | Build-time markdown resort data (T1/T2 tiers). |
| **UGC** | User-generated content: reviews, comments, passport photos. |
| **Saved resort** | User-bookmarked resort via heart icon; guest = device-local, user = synced. |
| **AI handoff** | Phase 2 data shapes and entry points Phase 3 AI consumes to generate trip content. |

---

## 4. Feature Priority & Build Sequence

Phase 2 is dependency-ordered. **Do not start community features without auth + database.**

| Priority | Epic theme | Rationale |
|----------|------------|-----------|
| **P0** | Auth + DB + roles + GDPR account flows | Foundation for all logged-in features |
| **P0** | SEO guardrails (public resort content) | Non-negotiable; embed in every story |
| **P1** | Resort map (Google Places) | High discovery value; low user friction; no login |
| **P1** | Reviews + comments + photos + moderation API | Core community; photos ship with reviews (not deferred) |
| **P1** | Ski Passport + badges | Engagement loop; complements reviews |
| **P1** | Trip skeleton UI + schema | Phase 3 AI dependency |
| **P1** | Saved resorts (heart) | Lightweight personalisation; Phase 3 AI context |
| **P1** | Public profile (`/u/:username`) | Badges, passport, reviews showcase |
| **P1** | Adaptive navigation (mobile bottom + desktop top) | Phase 2 surfaces need IA without crowding |
| **P2** | AI entry-point stubs (disabled) | Prepare Phase 3 without API cost |
| **P3** | Admin moderation UI | Deferred Phase 3; API sufficient at low traffic |
| **P3** | Review aggregate themes / AI summary | Phase 3 with FR-6 |
| **Future** | Slope account linking | Partnership TBD |

### Deferred / optional (not Phase 2 MVP)

| Feature | When |
|---------|------|
| **Notification opt-in (email)** | Phase 2.1 or Phase 3 |
| **Share trip skeleton (read-only link)** | Phase 2.1 |

---

## 5. Features

### 5.1 Authentication & Account Management (FR-8)

**Description:** Optional login via **Google OAuth** and **email magic link** (email + Google only — no Apple, no password in Phase 2). Login is **gated only** for: writing reviews, writing comments, trip planning, Ski Passport check-ins, syncing saved resorts across devices, and reporting content. Browsing, quiz, and all resort editorial content remain public without auth. Sessions persist on the same device with secure HTTP-only cookies; session timeout applies. Full GDPR/PDPA account export and deletion flows required.

Sign-in UX must **reduce magic-link anxiety** — many users equate "no password" with less security when the opposite is true. Offer Google alongside email so users can pick a familiar path; use trust-building copy and confirmation states (see addendum §A.1).

Realizes UJ-2, UJ-4, UJ-5.

#### FR-8.1: Sign in with Google or magic link

**User** can authenticate via Google OAuth or email magic link (one-time link, 15-minute expiry).

**Consequences (testable):**
- Sign-in sheet appears only when user attempts a gated action or taps "Sign in" in nav.
- Email path shows trust copy: secure one-time link, expiry time, spam-folder hint, masked recipient on confirmation (see addendum §A.1).
- Google OAuth offered as equal first-class option — not buried below email.
- Successful auth creates/links Supabase user; PostHog anonymous ID aliases to `user_id`.
- JWT stored in HTTP-only cookie — not `localStorage`.
- Session expires after **30-day rolling idle** timeout or **90-day absolute** maximum (whichever comes first).
- Magic link landing returns user to originating action (review draft, trip create, etc.).

#### FR-8.2: Guest browsing unchanged

**Guest** can browse all 20 resort detail pages, use quiz, search/filter Explore, **view public reviews and comments**, and **save resorts locally** without creating an account.

**Consequences:**
- Zero login walls on `/`, `/explore`, `/resort/[slug]`, `/quiz`.
- `is_logged_in: false` on analytics events for guests.

#### FR-8.3: Profile basics

**User** has display name, avatar (from Google or default), and optional short bio.

**Consequences:**
- Display name shown on reviews and comments (public identity — no anonymous mode).
- User can edit display name; server rejects display names matching the profanity blocklist (addendum §O).

#### FR-8.4: Account deletion & data export (GDPR/PDPA)

**User** can request account deletion and receive confirmation; **User** can export their data as JSON: profile, reviews, comments, trips, passport check-ins, badges, saved resorts.

**Consequences:**
- Deletion removes or anonymizes UGC per retention policy (see addendum).
- Privacy Policy and Terms updated before launch.
- Deletion completes within 30 days maximum; immediate lock-out on request.

#### FR-8.5: User roles

| Role | Capabilities |
|------|--------------|
| **Guest** | Browse resorts, use quiz, view public reviews, save resorts locally (device only) |
| **User** | Review, comment, create/edit/delete own trips (≤5), passport check-ins, save resorts (synced), report content |
| **Admin** | Hide/delete any review or comment; manage UGC via API/DB (UI Phase 3) |

#### FR-8.6: Admin role assignment

**System** assigns `admin` role only via manual update to `profiles.role` in Supabase (no self-service promotion).

**Consequences:**
- Initial admin(s) set during Supabase project setup.
- Admin API routes verify `profiles.role = admin` server-side.
- No admin UI Phase 2 — admins use API routes or Supabase dashboard.

**Feature-specific NFRs:**
- Rate limit auth endpoints: 10 attempts/min/IP.
- Email verification required before first UGC submission.
- OWASP-aligned: CSRF protection on mutations, secure cookie flags, Supabase RLS on all tables.

---

### 5.2 Resort Map & Surroundings (FR-9)

**Description:** Each resort detail page gains an embedded **Map** section showing the resort location plus nearby points of interest via **Google Places API** (restaurants, onsen, convenience, attractions). Map **renders** with **Leaflet + OpenStreetMap tiles** (Phase 1 stack); POI data from Google Places server proxy. Phase 1 trail map link-out remains. Map is public — no login.

Realizes UJ-1.

#### FR-9.1: Resort map section

**Guest or User** can view an interactive map centered on the resort with category-filtered nearby places.

**Consequences:**
- Map section title: **"Around the resort"** — placed **below Getting There**, above the Experience section (see addendum §M).
- Resort center from seed content `latitude` / `longitude` (addendum §K) — not geocoded at runtime.
- Categories: at minimum `restaurant`, `lodging`, `spa` (onsen), `tourist_attraction`, `convenience`.
- Pin tap shows name, rating (if available), short address; "Open in Google Maps" external link.
- Google Places API called server-side via proxy route — API key never exposed client-side.
- Google Places attribution displayed per ToS (addendum §L).
- Graceful fallback if Places API fails: static resort pin + link to Google Maps search.
- Map lazy-loaded (intersection observer) — NFR-16.

#### FR-9.2: Cost control

**System** caches Places results per resort per category with TTL (24h minimum).

**Consequences:**
- Estimated cost at 20 resorts × low traffic: well within Google $200/mo free credit `[ASSUMPTION]`.
- Analytics: `map_section_viewed`, `map_pin_tapped` with `place_category`.

**Out of Scope Phase 2:**
- Accommodation booking or Powri-curated lodging recommendations.
- Custom editorial POI overrides (Phase 3+ if needed).

---

### 5.3 Reviews & Comments (FR-10)

**Description:** Logged-in users submit **public reviews** (rating + text + optional photos) and **comments** on resort detail pages. Reviews show author profile (name + avatar + badges). No anonymous mode — encourages thoughtful, accountable contributions. Moderation: users report; admins hide/delete via API (admin UI Phase 3).

**Experience section layout (decided):** Single **"How people experienced it"** section on resort detail (addendum §M):
1. Aggregate rating + review count (SSR-friendly)
2. Review list (rating, text, photos, author profile link)
3. **Flat comment thread** below reviews — no nested replies in Phase 2
4. CTAs: "Write a review" / "Add a comment" (sign-in gated)

Realizes UJ-2, UJ-5.

#### FR-10.1: Submit review

**User** can submit one review per resort (editable after submit); includes 1–5 star rating, text (max 2,000 chars), up to **5 photos** (10 MB each).

**Consequences:**
- Review appears in **Experience** section on resort detail.
- Aggregate display: average rating + review count visible to **Guest and User**.
- Full review list visible to all (public reviews with identity).
- `review_submitted` analytics event fires on success.

#### FR-10.2: Comment on resort

**User** can post comments (max 500 chars) in a **flat comment thread** below the review list on resort detail — **no nested replies, no threading** in Phase 2.

**Consequences:**
- Comments show author profile + timestamp.
- Rate limit: 10 comments/hour/user.
- Comments are visually distinct from reviews (no star rating, lighter card style).

#### FR-10.3: Report review or comment

**User** (logged in only) can report a review or comment with reason enum (`spam`, `offensive`, `off_topic`, `other`).

**Consequences:**
- Guest tapping Report sees sign-in sheet with `trigger: report`.
- Report creates moderation queue record; reporter notified "Thank you — we'll review."
- Duplicate report same item by same user blocked.

#### FR-10.4: Admin moderation (API-level)

**Admin** can hide or delete reviews and comments via authenticated admin API or direct DB (no admin UI Phase 2).

**Consequences:**
- Hidden content not rendered for any user.
- Delete is soft-delete with audit log (admin_id, timestamp, action).
- Minimum requirements met: Guest/User report; Admin hide/delete.

**Out of Scope Phase 2:**
- AI-generated review theme summaries (Phase 3).
- Review voting / helpful counts (consider Phase 3).
- Anonymous reviews.

---

### 5.4 Ski Passport & Badges (FR-11)

**Description:** Gamified **Hall of Fame** — users manually check in to resorts they've visited, optionally attach a photo, and earn badges. Warm, playful tone. Public profile shows badge grid. Manual check-in only (no evidence required). Future Slope partnership noted but not in scope.

Realizes UJ-3.

#### FR-11.1: Manual check-in

**User** can tap **I've visited** on a resort detail page once per resort.

**Consequences:**
- Check-in recorded with `resort_slug`, `visited_at` (user-editable date optional `[ASSUMPTION: default today]`), optional note.
- User can undo check-in within 24 hours.
- One check-in per user per resort.

#### FR-11.2: Optional passport photo

**User** can attach up to 1 photo per check-in (10 MB max) displayed on passport entry.

**Consequences:**
- Photo stored in object storage (Supabase Storage); not required for check-in.
- Photo moderated same as review photos (report + admin hide).

#### FR-11.3: Badge awards

**System** evaluates badge rules on each check-in and awards badges automatically.

**Consequences:**
- Badge unlock triggers in-app toast + `badge_earned` analytics event.
- Badge definitions in addendum § Badge Catalog.
- Badges visible on user public profile and next to review author name.

#### FR-11.4: Passport view

**User** can view their Ski Passport: grid of visited resorts, badge shelf, optional stats (total resorts, regions).

**Consequences:**
- Own passport at `/passport` (logged in) or via hamburger menu.
- Public read-only view linked from public profile (FR-17).
- Guest can view others' public passports (encourages aspiration).

**Out of Scope Phase 2:**
- Slope account linking.
- Evidence verification (lift pass OCR, GPS).
- Leaderboards (consider if community stays healthy).

---

### 5.5 Trip Planning Skeleton (FR-12)

**Description:** Logged-in users create up to **5 trips** each. Trips use a **template itinerary** (placeholder activities) — not real accommodations or restaurants. Users can edit titles, notes, reorder activities, and delete trips to free quota. Schema designed for Phase 3 AI population. AI "Fill trip" button present as disabled stub with copy explaining Phase 3.

Realizes UJ-4.

#### FR-12.1: Create trip (quota 5)

**User** can create a new trip with: title, primary `resort_slug`, `start_date`, `end_date`, status (`draft` | `planned` | `completed`).

**Consequences:**
- Max **5 trips per user** at any time; user must delete a trip to create a 6th.
- Deletion is permanent (confirm dialog).
- `trip_created` analytics event.

#### FR-12.2: Template itinerary generation

**System** generates default `itinerary_days` from date range and resort template (see addendum § Trip Templates).

**Consequences:**
- Each day has ordered activities with types: `airport`, `transport`, `accommodation`, `ski`, `restaurant`, `attraction`, `custom`.
- Placeholders have generic titles ("Airport arrival", "Check in to accommodation", "Dinner nearby") — no real POI IDs.
- User can add/remove/reorder activities and edit title + notes.

#### FR-12.3: Trip list & detail views

**User** can view all trips in Plan tab/surface and open day-by-day itinerary view.

**Consequences:**
- Mobile-first vertical timeline UI.
- Empty state CTA: "Plan your first trip" with resort picker.
- Trip data persisted in PostgreSQL (not localStorage).

#### FR-12.4: AI handoff readiness

**System** exposes trip schema and `ai_ready: true` flag on trips with resort + dates set; stores `entry_point` context for Phase 3.

**Consequences:**
- Documented JSON schema in addendum § Trip Object Schema.
- Disabled **"Fill with AI"** CTA on trip detail — visible, explains "Coming in Phase 3".
- Entry points on quiz results page and resort detail: **"Plan this trip with AI"** stub (same disabled state).
- Quiz results **not** stored on user profile Phase 2; Phase 3 AI receives quiz answers only from in-session handoff or explicit re-take `[per PO decision]`.

**Out of Scope Phase 2:**
- Real hotel/restaurant/activity data or booking links in itinerary.
- Multi-resort trips (single primary resort Phase 2 `[ASSUMPTION]`).
- Sharing/editing trips with collaborators.
- AI population of itinerary.

---

### 5.6 Phase 3 AI Entry Points (FR-13)

**Description:** Visual stubs only — no live AI. Prepares user expectation and analytics baseline.

#### FR-13.1: Resort detail AI stub

**Guest or User** sees floating or inline **"Plan with AI"** affordance on resort detail (matches UX-DR17 placement from Phase 3 epic).

**Consequences:**
- Tap shows modal: "AI trip assistant coming soon — start a trip skeleton now" with CTA to FR-12 if logged in, or sign-in if not.
- `ai_entry_point_tapped` with `surface: resort_detail`, `ai_available: false`.

#### FR-13.2: Quiz results AI stub

**Guest or User** sees **"Want help planning?"** on quiz results podium view.

**Consequences:**
- Same modal behaviour; `surface: quiz_results`.
- Does not persist quiz result to profile.

---

### 5.7 SEO & Content Access (FR-14)

**Description:** Powri's organic traffic depends on public resort content. Logged-in features must not hide editorial content.

#### FR-14.1: Public resort content

**Guest** can access all resort editorial sections (hero, highlights, lowlights, terrain data, Getting There, trail map link) without login.

**Consequences:**
- No `noindex` on resort pages.
- Aggregate rating + review count visible to all users; full review bodies per FR-14.3.
- Trips and passport editing require login; trip content not indexed.
- Server-rendered resort pages remain SSG/ISR — editorial body in static HTML; UGC per FR-14.3.

#### FR-14.2: Auth-gated surfaces only

Login required **only** for: create/edit review, create/edit comment, report, trip CRUD, passport check-in, profile edit, cross-device saved-resort sync.

#### FR-14.3: SEO strategy for UGC (OQ-1 resolved)

**System** preserves crawlable editorial body while limiting SEO risk from thin/duplicate UGC pages.

**Consequences:**
- Resort detail **editorial sections** remain fully SSR/SSG in initial HTML (hero, highlights, Getting There, etc.).
- **Aggregate** rating + review count rendered in SSR HTML for rich-snippet eligibility.
- **Individual review text** loaded client-side after hydration (not duplicated in static HTML per review URL — no per-review routes in Phase 2).
- No `noindex` on resort pages.
- Public profile pages (`/u/:username`) use `noindex, follow` until review volume justifies index (revisit Phase 2.1).
- Trips remain private — never indexed.

---

### 5.8 Saved Resorts (FR-15)

**Description:** Users save resorts with a **heart** icon from resort cards and detail pages. Saved list is a first-class **Saved** destination in navigation. Feeds Phase 3 AI context (`saved_resorts` in trip handoff payload). Guests may save on-device; login merges local saves to account.

Realizes UJ-1 (resolution — Yuki hearts Niseko for later).

#### FR-15.1: Save / unsave resort

**Guest or User** can tap heart on resort card or detail page to save or unsave.

**Consequences:**
- **Guest:** saves persist in `localStorage` on current device only.
- **User:** saves persist in PostgreSQL; sync across devices.
- On first login after guest saves: merge localStorage slugs into account (dedupe); clear local copy after merge.
- Heart animates with subtle fill; active state visible on card and detail.
- `resort_saved` / `resort_unsaved` analytics with `resort_slug`, `is_logged_in`.

#### FR-15.2: Saved list view

**Guest or User** can open **Saved** from nav and see all saved resorts as resort cards (same card component as Explore).

**Consequences:**
- Route: `/saved` — public route, no login wall; empty state CTA to Explore.
- Guest sees banner: "Sign in to sync saved resorts across devices" (dismissible).
- Sort: most recently saved first.

**Out of Scope Phase 2:**
- Saved search queries or filters.
- Sharing saved list with others.

---

### 5.9 Information Architecture & Navigation (FR-16)

**Description:** Phase 2 adds Saved, Plan, and Passport without crowding mobile or feeling like a "fake app" on desktop web. **Adaptive navigation:** bottom tab bar on mobile, horizontal top nav on desktop; secondary destinations in hamburger menu.

**Decision:** Do **not** remove bottom nav on mobile — Powri remains mobile-first (375–430px primary) and thumb-zone navigation still wins on mobile web. Do **not** put 5+ items in bottom nav — use **4 primary tabs + hamburger overflow**. On desktop (≥768px), **hide bottom nav** and show top nav instead — this addresses web-only rollout feeling.

#### FR-16.1: Mobile navigation (<768px)

**Guest or User** sees fixed bottom nav with **4 items:** Home · Explore · Saved · Plan.

**Consequences:**
- Bottom nav always visible; never hide on scroll (Phase 1 behaviour preserved).
- **Passport**, **Info**, **Account / Sign in** live in **hamburger menu** (top bar — extend existing `AppMenuSheet`).
- Plan tab: logged-out users see empty state + sign-in CTA; no login wall on tab itself.
- Active state styling matches Phase 1 Theme A.

#### FR-16.2: Desktop navigation (≥768px)

**Guest or User** sees **horizontal top nav** instead of bottom nav: Home · Explore · Saved · Plan · Passport · Info · Sign in / Avatar.

**Consequences:**
- Bottom nav hidden at **`md` breakpoint (768px)** and above — **OQ-3 resolved**.
- Top nav in header bar; sticky on scroll.
- Same routes as mobile — no desktop-only dead ends.

#### FR-16.3: Hamburger menu (mobile overflow)

**Guest or User** opens hamburger from top bar to reach Passport, Info, Account, Sign in/out.

**Consequences:**
- Extend `AppMenuSheet` nav items — do not duplicate bottom-nav destinations in hamburger (Home/Explore/Saved/Plan stay in bottom bar only).
- Passport shows badge count chip if user has ≥1 badge.
- `nav_menu_opened` event retained; add `nav_destination` property on nav taps.

**Rationale (vs alternatives considered):**

| Option | Verdict |
|--------|---------|
| Hamburger-only (no bottom nav) | Rejected — extra tap for core flows on mobile; hurts discovery |
| 5-item bottom nav (add Passport) | Rejected — crowded at 375px; icon labels truncate |
| Bottom nav on desktop too | Rejected — feels wrong on web; users expect top nav |
| **4-tab mobile + top desktop + hamburger overflow** | **Selected** — scales Phase 2 without UX debt |

---

### 5.10 Public Profile (FR-17)

**Description:** Each user has a **public profile page** showcasing their community presence — badges, passport grid, and published reviews. Supports the "warm, accountable identity" goal for reviews and passport sharing.

Realizes UJ-3 (resolution — share passport URL).

#### FR-17.1: Public profile page

**Guest or User** can view `/u/:username` showing display name, avatar, badge shelf, visited-resort grid, and list of public reviews (resort name + rating + excerpt).

**Consequences:**
- Username slug derived from display name (unique, URL-safe; collision suffix `-2`, etc.).
- No email or private data exposed.
- Owner can edit profile from `/account` (logged in).
- `noindex, follow` meta on profile pages Phase 2 (FR-14.3).
- Links from review author name → public profile.

#### FR-17.2: Account settings

**User** can access `/account` to edit display name, bio, avatar; export data; request deletion; sign out.

**Consequences:**
- Linked from hamburger / avatar menu.
- Deletion and export per FR-8.4.

---

## 6. Cross-Cutting NFRs

| ID | Requirement |
|----|-------------|
| **NFR-10** | All UGC sanitized on input; HTML stripped; DOMPurify on render. |
| **NFR-11** | Supabase Row Level Security: users read/write own trips/passport; public read on published reviews. |
| **NFR-12** | Photo upload: virus scan or MIME validation; max 5 photos/review, 1/check-in; 10 MB each; WebP/JPEG/PNG only. |
| **NFR-13** | Session idle timeout + absolute max; secure cookie flags. |
| **NFR-14** | Account deletion completes within 30 days; immediate session revocation on request. |
| **NFR-15** | Google Places + Supabase keys server-side only. |
| **NFR-16** | Core Web Vitals on resort detail with map: no regression vs Phase 1 (map lazy-loaded). |
| **NFR-17** | Adaptive nav: mobile bottom **Home · Explore · Saved · Plan**; desktop top nav adds Passport · Info · Account; hamburger overflow on mobile. |

---

## 7. Data Architecture Summary

PostHog is **analytics only** — not a user content database. Phase 2 requires:

| Layer | Technology | Purpose |
|-------|------------|---------|
| Database | Supabase PostgreSQL | Users, reviews, comments, trips, passport, saved resorts, reports |
| Auth | Supabase Auth | Google OAuth + email |
| File storage | Supabase Storage | Review + passport photos |
| Analytics | PostHog (EU) | Events, funnels — unchanged pattern |
| Resort content | Markdown (build-time) | Unchanged — no UGC in markdown |

Full schema: [`addendum.md`](addendum.md).

---

## 8. Non-Goals (Explicit)

- Apple Sign-In (Phase 2) — Google + email only to reduce setup friction.
- Anonymous reviews or comments.
- Storing quiz results on user profile (Phase 2).
- Live AI chat or trip generation (Phase 3).
- Admin moderation UI (Phase 3).
- Real accommodation/restaurant inventory in trips (Phase 3 AI + optional integrations).
- Slope partnership / account linking (future).
- Monetisation / affiliate (unchanged deferral).
- TH/JA full locales (scaffold only).
- Evidence-based passport check-in.

---

## 9. MVP Scope (Phase 2)

### 9.1 In Scope

- Supabase auth (Google + email), roles, GDPR deletion/export
- Resort map with Google Places proxy + cache
- Reviews, comments, report, admin API moderation
- Ski Passport check-in, badges, optional photos
- Trip skeleton (5 max), template itinerary, edit UI
- Saved resorts (heart + `/saved` list; guest localStorage + account sync)
- Adaptive navigation (4-tab mobile bottom nav + desktop top nav)
- Public profile (`/u/:username`) + account settings
- AI entry-point stubs (disabled)
- SEO public content guardrails
- Analytics events (see §10)
- Privacy/Terms update for UGC + accounts

### 9.2 Out of Scope for Phase 2

| Item | Reason / When |
|------|----------------|
| AI trip fill | Phase 3 |
| Admin UI | Phase 3 — low traffic; API/DB moderation sufficient |
| Review AI summaries | Phase 3 |
| Apple auth | PO decision — cost/complexity |
| Multi-resort trips | Simplify Phase 2 skeleton |
| Email notifications | Optional P2 — not committed |

---

## 10. Success Metrics

### 10.1 Business Success Criteria

| ID | Metric | Target (90 days post Phase 2 launch) | Notes |
|----|--------|--------------------------------------|-------|
| **BS-1** | Registered users | ≥ 500 | Organic + quiz funnel |
| **BS-2** | Weekly active users (WAU) | ≥ 30% of registered | Return intent |
| **BS-3** | UGC participation rate | ≥ 15% of WAU submit ≥1 review or check-in | Community health |
| **BS-4** | Trip skeleton creation | ≥ 20% of registered users create ≥1 trip | Phase 3 AI funnel |
| **BS-5** | SEO stability | Organic resort page traffic ≥ 90% of Phase 1 baseline | FR-14 guardrail |
| **BS-6** | Account deletion SLA | 100% within 30 days | Compliance |

### 10.2 Feature Analytics Success Criteria

| ID | Metric | Target | Validates |
|----|--------|--------|-----------|
| **SM-1** | Map engagement | ≥ 25% of `resort_detail_viewed` also fire `map_section_viewed` | FR-9 |
| **SM-2** | Review submission rate | ≥ 5% of logged-in resort viewers submit review | FR-10 |
| **SM-3** | Badge earn rate | ≥ 40% of users with ≥1 check-in earn ≥2 badges | FR-11 |
| **SM-4** | Trip quota usage | Average 1.5 trips/user among trip creators | FR-12 |
| **SM-5** | AI stub tap-through | ≥ 10% of stub taps → sign-in or trip create | FR-13 Phase 3 funnel |
| **SM-6** | Auth conversion | ≥ 35% of sign-in sheet opens complete auth | FR-8 |
| **SM-7** | Report rate | < 2% of reviews reported (quality signal) | FR-10 |
| **SM-8** | Saved resort adoption | ≥ 20% of WAU save ≥1 resort | FR-15 |
| **SM-9** | Magic-link completion | ≥ 30% of magic-link sends → click within 15 min | FR-8 |

### 10.3 Counter-Metrics (do not optimize)

| ID | Metric | Why |
|----|--------|-----|
| **SM-C1** | Raw sign-up count without WAU | Vanity metric — prefer BS-2 |
| **SM-C2** | Check-ins without visits | Discourage fake badge farming — monitor anomalies |
| **SM-C3** | Trip count without edit events | Empty templates don't predict Phase 3 AI success |

### 10.4 New Analytics Events (add to tracking-plan)

| Event | Trigger | Key properties |
|-------|---------|----------------|
| `auth_sign_in_started` | Sign-in sheet opened | `provider` (`google`/`email`), `trigger` (`review`/`trip`/`passport`/`nav`) |
| `auth_sign_in_completed` | Successful auth | `provider`, `is_new_user` |
| `auth_sign_out` | User signs out | — |
| `account_deletion_requested` | User requests deletion | — |
| `map_section_viewed` | Map enters viewport | `resort_slug` |
| `map_pin_tapped` | Place pin tapped | `resort_slug`, `place_category` |
| `review_submitted` | Review posted | `resort_slug`, `rating`, `photo_count` |
| `comment_submitted` | Comment posted | `resort_slug` |
| `content_reported` | Report filed | `content_type` (`review`/`comment`), `reason` |
| `passport_check_in` | Manual visit recorded | `resort_slug` |
| `badge_earned` | Badge unlocked | `badge_id` |
| `trip_created` | New trip saved | `resort_slug`, `day_count` |
| `trip_deleted` | Trip removed | `trip_count_remaining` |
| `trip_activity_edited` | Activity title/notes changed | `activity_type` |
| `ai_entry_point_tapped` | AI stub clicked | `surface`, `ai_available` |
| `resort_saved` | Heart toggled on | `resort_slug`, `surface` (`card`/`detail`), `is_logged_in` |
| `resort_unsaved` | Heart toggled off | `resort_slug`, `surface`, `is_logged_in` |
| `saved_list_viewed` | `/saved` page opened | `saved_count`, `is_logged_in` |
| `magic_link_sent` | Email link requested | — |
| `magic_link_completed` | User lands from email link | `is_new_user` |
| `nav_destination_tapped` | Nav item tapped | `destination`, `nav_type` (`bottom`/`top`/`hamburger`) |

---

## 11. Resolved Decisions (formerly open questions)

| # | Question | Decision | Reference |
|---|----------|----------|-----------|
| ~~OQ-1~~ | Reviews indexable by Google — full text or aggregate? | **Aggregate rating + count in SSR HTML; review bodies client-hydrated; no per-review URLs; profiles `noindex`** | FR-14.3 |
| ~~OQ-2~~ | Flat comments vs one-level replies? | **Flat comment thread only — no nesting in Phase 2** | FR-10.2 |
| ~~OQ-3~~ | Desktop breakpoint 768 vs 1024? | **`md` = 768px** — hide bottom nav, show top nav | FR-16.2 |

### Remaining open items (non-blocking for PRD final)

| # | Question | Owner | Revisit |
|---|----------|-------|---------|
| OQ-4 | Review author anonymization on account delete — legal confirm | Legal | Before UGC launch |
| OQ-5 | Public profile `noindex` lift threshold (e.g. N reviews site-wide) | PM + SEO | Phase 2.1 |

---

## 12. Assumptions Index

- Email auth: magic link only (no password); Google OAuth as parallel option.
- Magic-link trust UX follows addendum §A.1 copy patterns.
- Navigation: 4-tab mobile bottom nav + desktop top nav at **768px** + hamburger overflow (FR-16).
- Saved resorts: guest localStorage + merge on login.
- Session: **30-day rolling idle**, **90-day absolute** max (FR-8.1).
- Single-resort trips only in Phase 2.
- **Flat comments only** — no nested replies (OQ-2 closed).
- **SEO:** aggregate in SSR, review bodies client-side (OQ-1 closed).
- Logged-in required to report content.
- Google Places free tier sufficient at 20-resort scale.
- Map render: Leaflet + OSM tiles; POIs from Google Places proxy.
- Admin role: manual `profiles.role` assignment only (FR-8.6).
- Profanity: server-side blocklist on display names (addendum §O).
- Badge regions: derived from seed content `region` field (addendum §C.1).
- Ski season for badges: **1 Nov – 30 Apr JST** (addendum §C.2).
- Admin moderation via API/Supabase dashboard until Phase 3 UI.

---

## 13. Release Gates (Phase 2)

- [ ] Pre-implementation checklist complete ([`readiness-checklist.md`](readiness-checklist.md) §A–D)
- [ ] All 20 resorts have `latitude` / `longitude` in seed content
- [ ] Supabase RLS policies tested (user cannot read others' draft trips)
- [ ] GDPR deletion flow end-to-end tested
- [ ] Google Places proxy rate-limited and cached
- [ ] Google Places attribution visible on map UI
- [ ] All new analytics events verified in PostHog (`docs/tracking-plan.md` synced)
- [ ] Privacy Policy + Terms + community guidelines updated for UGC
- [ ] Resort pages remain crawlable (SSR/SSG editorial body)
- [ ] `npm run lint && npm run build && npm run test:launch` pass
- [ ] Photo upload limits enforced client + server
- [ ] Moderation report → admin hide path tested manually

---

## 14. Downstream Artifacts Required Before Implementation

Do **not** start Sprint 1 code until these exist. Full checklist: [`readiness-checklist.md`](readiness-checklist.md).

| Artifact | Skill / action | Blocks |
|----------|--------------|--------|
| Phase 2 architecture delta | `bmad-create-architecture` | P0 auth, all API routes |
| Phase 2 epics & stories | `bmad-create-epics-and-stories` | All implementation |
| Phase 2 UX spec (screens) | `bmad-ux` | UI stories |
| Tracking plan §3 update | Edit `docs/tracking-plan.md` | Analytics per story |
| Content schema + lat/lng | Edit `CONTENT_MODEL.md` + 20 resorts | Map FR-9 |
| Privacy / Terms / guidelines | Legal + PM | UGC public launch |

---

*PRD revision 3 patches inconsistencies and closes OQ-1–3. Next: complete [`readiness-checklist.md`](readiness-checklist.md) §B–D, then `bmad-create-architecture` → `bmad-create-epics-and-stories`.*
