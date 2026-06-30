---
title: Powri Phase 2 — Epics & Stories
created: 2026-06-23
updated: 2026-06-23
status: approved
phase: 2
inputDocuments:
  - prds/prd-japan_winter_sport-2026-06-23/prd.md
  - prds/prd-japan_winter_sport-2026-06-23/addendum.md
  - architecture-phase2.md
  - supabase/migrations/001_phase2_core.sql
project_name: japan_winter_sport
story_range: 9.1–15.4
---

# Phase 2 Epics & Stories

**Powri Phase 2** — auth, saved resorts, map, reviews, passport, trip skeleton, launch gates.

**PRD:** [`prds/prd-japan_winter_sport-2026-06-23/prd.md`](../prds/prd-japan_winter_sport-2026-06-23/prd.md)  
**Architecture:** [`architecture-phase2.md`](../architecture-phase2.md)  
**Shards:** [`epic-09`](epic-09-supabase-auth.md) · [`epic-10`](epic-10-nav-saved.md) · [`epic-11`](epic-11-resort-map.md) · [`epic-12`](epic-12-community-reviews.md) · [`epic-13`](epic-13-passport-profile.md) · [`epic-14`](epic-14-trip-planning.md) · [`epic-15`](epic-15-phase2-launch.md)

---

## Requirements Coverage Map (Phase 2)

| FR | Epic | Stories |
|----|------|---------|
| FR-8 Auth & account | Epic 9 | 9.1–9.5 |
| FR-16 Navigation | Epic 10 | 10.1–10.2 |
| FR-15 Saved resorts | Epic 10 | 10.3–10.4 |
| FR-9 Resort map | Epic 11 | 11.1–11.3 |
| FR-10 Reviews & comments | Epic 12 | 12.1–12.4 |
| FR-11 Ski Passport & badges | Epic 13 | 13.1–13.2 |
| FR-17 Public profile | Epic 13 | 13.3–13.4 |
| FR-12 Trip skeleton | Epic 14 | 14.1–14.3 |
| FR-13 AI stubs | Epic 15 | 15.1 |
| FR-14 SEO | Epic 15 | 15.4 |
| NFR-10–17 | Epics 9–15 | per story |

---

## Epic List

| Epic | Title | User outcome | Stories |
|------|-------|--------------|---------|
| **9** | Supabase Foundation & Auth | Sign in with Google or magic link; manage account | 9.1–9.5 |
| **10** | Navigation & Saved Resorts | Find Plan/Saved in nav; heart resorts | 10.1–10.4 |
| **11** | Resort Map & Surroundings | See nearby POIs on resort detail | 11.1–11.3 |
| **12** | Community Reviews & Comments | Share and read experiences | 12.1–12.4 |
| **13** | Ski Passport & Public Profile | Check in, earn badges, public profile | 13.1–13.4 |
| **14** | Trip Planning Skeleton | Create template itineraries (max 5) | 14.1–14.3 |
| **15** | Phase 2 Launch & Analytics | Stubs, tracking, legal, release gates | 15.1–15.4 |

**Implementation order:** 9 → 10 → 11 ∥ 12 (after 9) → 13 → 14 → 15. Epic 11 blocked until lat/lng content (11.1).

---

## Epic 9: Supabase Foundation & Auth

Users can sign in with Google or email magic link, manage their profile, and export or delete their account.

**FRs:** FR-8 · **Architecture:** §3–4 · **Shard:** [`epic-09-supabase-auth.md`](epic-09-supabase-auth.md)

### Story 9.1: Supabase Project & Database Migration

As a **developer**,  
I want Supabase provisioned with Phase 2 schema and RLS,  
So that all UGC features have a secure data foundation.

**Acceptance Criteria:**

**Given** a Supabase project is created  
**When** migration [`001_phase2_core.sql`](../../../supabase/migrations/001_phase2_core.sql) is applied  
**Then** all tables exist: `profiles`, `reviews`, `review_photos`, `comments`, `passport_check_ins`, `user_badges`, `trips`, `trip_days`, `trip_activities`, `saved_resorts`, `content_reports`, `moderation_audit`, `places_cache`

**And** RLS is enabled on user-facing tables per migration  
**And** Storage buckets `review-photos` and `passport-photos` exist  
**And** `web/.env.example` vars documented in Vercel preview + production

---

### Story 9.2: Supabase SSR Clients & Session Middleware

As a **returning user**,  
I want my session to persist securely across page loads,  
So that I stay signed in without storing tokens in localStorage.

**Acceptance Criteria:**

**Given** `@supabase/supabase-js` and `@supabase/ssr` are installed  
**When** I navigate the app  
**Then** `lib/supabase/client.ts`, `server.ts`, and `middleware.ts` exist per architecture-phase2 §4.2

**And** middleware calls session refresh before intl routing (after `/ingest` bypass)  
**And** JWT is stored in HTTP-only cookies only  
**And** `getCommonProperties()` sets `is_logged_in: true` when session exists

---

### Story 9.3: Sign-In Sheet — Google OAuth & Magic Link

As a **guest**,  
I want to sign in with Google or a secure email link when I choose to contribute,  
So that I can review, plan, or check in without a password.

**Acceptance Criteria:**

**Given** I tap a gated action (review, comment, trip, check-in, report) or Sign in in nav  
**When** the sign-in sheet opens  
**Then** **Continue with Google** is the primary button; email magic link is secondary (addendum §A.1)

**And** email path shows trust copy, expiry, spam hint; confirmation shows masked email  
**And** `/api/auth/callback` handles OAuth and magic-link return  
**And** magic link lands user on originating action via `returnTo`  
**And** `auth_sign_in_started` and `auth_sign_in_completed` fire with `provider` and `is_new_user`  
**And** PostHog aliases anonymous ID to user ID on first completion  
**And** rate limit `auth` policy: 10/min/IP

---

### Story 9.4: Profile Bootstrap & Account Settings

As a **signed-in user**,  
I want a profile with display name and avatar and an account settings page,  
So that my public identity appears on reviews and comments.

**Acceptance Criteria:**

**Given** I complete first sign-in  
**When** my profile is created via DB trigger  
**Then** I can set a unique `username` (URL slug) and edit display name, bio, avatar on `/account`

**And** display name rejects profanity blocklist server-side  
**And** default username prompt if still `user-{id}` pattern  
**And** Admin role is **not** self-service — manual DB only (FR-8.6)

---

### Story 9.5: GDPR Data Export & Account Deletion

As a **signed-in user**,  
I want to export my data or delete my account,  
So that I control my personal information under GDPR/PDPA.

**Acceptance Criteria:**

**Given** I am on `/account`  
**When** I request export  
**Then** I receive JSON download: profile, reviews, comments, trips, check-ins, badges, saved resorts

**When** I request deletion  
**Then** I am signed out immediately; account locked; deletion completes within 30 days  
**And** `account_deletion_requested` event fires  
**And** Privacy Policy link updated (coordinate with Story 15.3)

---

## Epic 10: Navigation & Saved Resorts

Users discover Plan and Saved from navigation; guests can heart resorts locally; signed-in users sync saves.

**FRs:** FR-15, FR-16 · **Shard:** [`epic-10-nav-saved.md`](epic-10-nav-saved.md)

### Story 10.1: Adaptive Navigation — Mobile Bottom + Desktop Top

As a **visitor on any device**,  
I want clear navigation to Home, Explore, Saved, and Plan,  
So that I find Phase 2 features without hunting menus.

**Acceptance Criteria:**

**Given** viewport **< 768px**  
**When** I view any main page  
**Then** bottom nav shows **Home · Explore · Saved · Plan** (4 items, always visible)

**Given** viewport **≥ 768px**  
**Then** bottom nav is hidden; **DesktopTopNav** shows Home · Explore · Saved · Plan · Passport · Info · Sign in/Avatar

**And** active states match Theme A  
**And** `nav_destination_tapped` fires with `destination` and `nav_type`

---

### Story 10.2: Hamburger Overflow Menu

As a **mobile user**,  
I want Passport, Info, and Account in the hamburger menu,  
So that secondary destinations don't crowd the bottom bar.

**Acceptance Criteria:**

**Given** viewport < 768px  
**When** I open the hamburger  
**Then** I see **Passport**, **Info**, **Sign in / Account** — **not** Home/Explore/Saved/Plan (no duplication)

**And** Passport shows badge count chip when user has ≥1 badge  
**And** `nav_menu_opened` retained from Phase 1

---

### Story 10.3: Save Resort Heart — Guest & Authenticated

As a **visitor**,  
I want to heart resorts from cards and detail pages,  
So that I can shortlist resorts for later.

**Acceptance Criteria:**

**Given** I tap heart on a resort card or detail  
**When** I am a **guest**  
**Then** slug saves to `localStorage` key `powri_saved_resorts`

**When** I am **signed in**  
**Then** slug saves to `saved_resorts` table via API

**And** `resort_saved` / `resort_unsaved` fire with `resort_slug`, `surface`, `is_logged_in`  
**And** heart shows active state on card and detail consistently

---

### Story 10.4: Saved List Page & Login Merge

As a **user**,  
I want a Saved page listing my hearted resorts and sync on login,  
So that my shortlist follows me across devices.

**Acceptance Criteria:**

**Given** I open `/saved`  
**When** I have saved resorts (local or synced)  
**Then** I see resort cards sorted by most recently saved

**And** guests see dismissible banner: sign in to sync across devices  
**And** on login, `POST /api/saved/merge` dedupes localStorage into DB and clears local key  
**And** `saved_list_viewed` fires with `saved_count`, `is_logged_in`

---

## Epic 11: Resort Map & Surroundings

Users see an interactive map of the resort and nearby places on each resort detail page.

**FRs:** FR-9 · **Shard:** [`epic-11-resort-map.md`](epic-11-resort-map.md)

### Story 11.1: Resort Geo Fields in Content Pipeline

As a **developer**,  
I want latitude and longitude on every published resort,  
So that the map centers correctly without runtime geocoding.

**Acceptance Criteria:**

**Given** `CONTENT_MODEL.md` and `lib/content/schema.ts` are updated  
**When** the build runs  
**Then** published resorts require `latitude` and `longitude` (optional `map_default_zoom`, default 13)

**And** all **20** published resort markdown files have valid coordinates  
**And** build warns on missing coords; fails once PO confirms all populated

---

### Story 11.2: Google Places Proxy & Cache

As the **system**,  
I want Places API calls server-side with caching,  
So that POI data stays secure and cost-bounded.

**Acceptance Criteria:**

**Given** `GOOGLE_PLACES_API_KEY` is server-only  
**When** `GET /api/places/nearby?resortSlug=&category=` is called  
**Then** lat/lng come from build-time resort index (not client params)  
**And** results cache in `places_cache` for 24h per resort+category  
**And** response includes minimal fields + Google attribution string  
**And** rate limit `places`: 60/min/IP  
**And** graceful empty response if API fails

---

### Story 11.3: Resort Map Section UI

As a **visitor on resort detail**,  
I want an "Around the resort" map with category pins,  
So that I discover nearby food, onsen, and attractions.

**Acceptance Criteria:**

**Given** I scroll to the map section (below Getting There, above Experience)  
**When** the section enters viewport  
**Then** `map_section_viewed` fires  
**And** Leaflet + OSM tiles render via dynamic import (ssr: false)  
**And** category filters: restaurant, lodging, spa, tourist_attraction, convenience  
**And** pin tap shows name, rating, vicinity; **Open in Google Maps** link  
**And** **Powered by Google** attribution visible  
**And** `map_pin_tapped` fires with `place_category`  
**And** CSP allows OSM tile domains

---

## Epic 12: Community Reviews & Comments

Signed-in users publish reviews and comments; anyone can read; admins can hide reported content.

**FRs:** FR-10 · **Shard:** [`epic-12-community-reviews.md`](epic-12-community-reviews.md)

### Story 12.1: Experience Section Shell & Review List

As a **visitor on resort detail**,  
I want to see aggregate rating and community reviews,  
So that I trust peer experiences before my trip.

**Acceptance Criteria:**

**Given** I view resort detail  
**When** the Experience section renders  
**Then** section title is **How people experienced it** (addendum §M order)

**And** aggregate ★ rating + count in SSR HTML (FR-14.3)  
**And** review list loads client-side from `/api/reviews?resortSlug=`  
**And** each review shows author name, avatar, badges link to `/u/:username`  
**And** guests see reviews without login wall

---

### Story 12.2: Submit & Edit Review with Photos

As a **signed-in visitor**,  
I want to write one review per resort with optional photos,  
So that I can share my experience publicly.

**Acceptance Criteria:**

**Given** I am signed in on resort detail  
**When** I submit a review (1–5 stars, text ≤2000 chars, ≤5 photos × 10MB)  
**Then** review saves with my profile identity (no anonymous mode)  
**And** I can edit my existing review for that resort  
**And** photos upload to `review-photos` bucket via signed URL flow  
**And** UGC sanitized on input (NFR-10)  
**And** `review_submitted` fires with `rating`, `photo_count`

---

### Story 12.3: Flat Comment Thread

As a **signed-in visitor**,  
I want to add quick comments below reviews,  
So that I can share shorter thoughts.

**Acceptance Criteria:**

**Given** the Experience section  
**When** I post a comment (≤500 chars)  
**Then** it appears in a **flat list** below reviews — no nesting (OQ-2)  
**And** comments visually distinct from reviews (no stars)  
**And** rate limit 10 comments/hour/user  
**And** `comment_submitted` fires

---

### Story 12.4: Report Content & Admin Hide API

As a **signed-in user**,  
I want to report inappropriate reviews or comments,  
So that the community stays respectful.

**Acceptance Criteria:**

**Given** I am signed in  
**When** I report content with reason enum  
**Then** row created in `content_reports`; duplicate blocked  
**And** `content_reported` fires

**Given** an admin with `role = admin`  
**When** they call `PATCH /api/admin/reviews/:id/hide` or comments equivalent  
**Then** content `status` becomes `hidden`; audit row in `moderation_audit`  
**And** hidden content not rendered for any user

**And** guests tapping Report see sign-in sheet (`trigger: report`)

---

## Epic 13: Ski Passport & Public Profile

Users check in to resorts, earn badges, and share a public profile.

**FRs:** FR-11, FR-17 · **Shard:** [`epic-13-passport-profile.md`](epic-13-passport-profile.md)

### Story 13.1: Manual Check-In & Optional Photo

As a **signed-in skier**,  
I want to mark resorts I've visited,  
So that my passport records my Japan ski journey.

**Acceptance Criteria:**

**Given** I am on resort detail, signed in  
**When** I tap **I've visited**  
**Then** check-in recorded with `resort_slug`, `visited_at` (default today)  
**And** one check-in per user per resort; undo within 24h  
**And** optional photo (1 × 10MB) to `passport-photos` bucket  
**And** `passport_check_in` fires

---

### Story 13.2: Badge Evaluation & Unlock Toast

As a **signed-in skier**,  
I want to earn playful badges automatically,  
So that visiting and reviewing feels rewarding.

**Acceptance Criteria:**

**Given** I check in or publish a review  
**When** badge rules evaluate (`lib/badges/evaluate.ts`, addendum §C)  
**Then** new badges insert idempotently into `user_badges`  
**And** in-app toast on unlock; `badge_earned` with `badge_id`  
**And** regional badges use seed content `region` mapping (§C.1)  
**And** season rules use JST Nov–Apr (§C.2)

---

### Story 13.3: Passport View

As a **signed-in user**,  
I want to see my visited resorts and badges,  
So that I can track my progress.

**Acceptance Criteria:**

**Given** I open `/passport` or hamburger → Passport  
**When** the page loads  
**Then** I see visited resort grid, badge shelf, stats (total resorts)

**And** mobile access via hamburger; desktop via top nav

---

### Story 13.4: Public Profile Page

As a **visitor**,  
I want to view another user's public profile,  
So that I can see their badges and reviews.

**Acceptance Criteria:**

**Given** I open `/u/:username`  
**When** the profile exists  
**Then** I see display name, avatar, bio, badge grid, passport resorts, review excerpts  
**And** no email or private data  
**And** `noindex, follow` meta tag (FR-14.3)  
**And** review author links navigate here

---

## Epic 14: Trip Planning Skeleton

Signed-in users create up to 5 template trip itineraries for a single resort.

**FRs:** FR-12 · **Shard:** [`epic-14-trip-planning.md`](epic-14-trip-planning.md)

### Story 14.1: Trip CRUD & Template Generation

As a **signed-in planner**,  
I want to create a trip with dates and a pre-filled day-by-day template,  
So that I have structure before AI helps in Phase 3.

**Acceptance Criteria:**

**Given** I am signed in  
**When** I create a trip (title, resort, start/end dates)  
**Then** `lib/trips/templates.ts` generates itinerary days and placeholder activities  
**And** max **5 trips** per user; 6th blocked with friendly error  
**And** delete frees quota with confirm dialog  
**And** `ai_ready: true` when resort + dates set  
**And** `trip_created` fires with `day_count`

---

### Story 14.2: Plan Tab — Trip List & Empty States

As a **visitor or user**,  
I want a Plan tab showing my trips or a sign-in CTA,  
So that trip planning has a clear home.

**Acceptance Criteria:**

**Given** I open `/plan` from bottom nav  
**When** I am logged out  
**Then** empty state + sign-in CTA — no login wall on tab itself

**When** I am logged in with trips  
**Then** I see trip cards with title, resort, dates, status

**When** I have no trips  
**Then** CTA **Plan your first trip** with resort picker

---

### Story 14.3: Trip Detail Timeline & Activity Edit

As a **signed-in planner**,  
I want to edit my itinerary days and activities,  
So that the skeleton matches my rough plan.

**Acceptance Criteria:**

**Given** I open `/plan/[tripId]`  
**When** I view the timeline  
**Then** vertical day-by-day layout with ordered activities (types per FR-12)

**And** I can edit title and notes, add/remove/reorder activities  
**And** **Fill with AI** button visible but disabled with Phase 3 tooltip  
**And** `trip_activity_edited` fires on save  
**And** `trip_deleted` fires on delete

---

## Epic 15: Phase 2 Launch, Analytics & Compliance

Phase 2 ships with AI stubs, analytics, legal updates, and release verification.

**FRs:** FR-13, FR-14, FR-7 (update) · **Shard:** [`epic-15-phase2-launch.md`](epic-15-phase2-launch.md)

### Story 15.1: Phase 3 AI Entry Point Stubs

As a **visitor**,  
I want to see where AI help is coming,  
So that I'm not surprised when Phase 3 launches.

**Acceptance Criteria:**

**Given** quiz results or resort detail  
**When** I tap **Want help planning?** or **Plan with AI**  
**Then** modal explains Phase 3; CTA to create trip skeleton or sign in  
**And** `ai_entry_point_tapped` with `surface`, `ai_available: false`  
**And** no live `/api/chat` product calls

---

### Story 15.2: Phase 2 Analytics Registry & Tracking Plan

As a **product owner**,  
I want all Phase 2 events instrumented and documented,  
So that we measure adoption from launch.

**Acceptance Criteria:**

**Given** PRD §10.4 event list  
**When** `phase2Events.ts` registry and `docs/tracking-plan.md` §3 are updated  
**Then** all Phase 2 events match canonical names (addendum §P deprecations)  
**And** `npm run test:analytics` extended or `test:analytics:phase2` added  
**And** events verified in PostHog before story sign-off

---

### Story 15.3: Privacy, Terms & Community Guidelines

As a **product owner**,  
I want legal pages updated for UGC and auth,  
So that we comply before community features go public.

**Acceptance Criteria:**

**Given** Phase 2 UGC ships  
**When** I read `/privacy` and `/terms`  
**Then** they cover: magic-link email, Google OAuth, photos, reviews, account deletion, data export

**And** short community guidelines accessible from review form or Info  
**And** moderation contact uses `NEXT_PUBLIC_CONTACT_EMAIL`

---

### Story 15.4: Phase 2 Release Gates & SEO Verification

As a **release owner**,  
I want Phase 2 launch gates verified,  
So that SEO and quality don't regress.

**Acceptance Criteria:**

**Given** PRD §13 release gates  
**When** QA runs pre-merge checklist  
**Then** RLS tested; GDPR deletion E2E; Places proxy cached; resort pages crawlable  
**And** editorial body in SSG HTML; review bodies client-hydrated only  
**And** `npm run lint && npm run build && npm run test:launch` pass  
**And** moderation report → admin hide path tested manually

---

*Phase 2 epics approved 2026-06-23. Total: 7 epics, 27 stories (9.1–15.4).*
