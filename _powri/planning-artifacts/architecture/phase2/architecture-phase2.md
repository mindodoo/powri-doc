---
title: Powri Phase 2 — Architecture Delta
created: 2026-06-23
updated: 2026-06-23
status: complete
phase: 2
builds_on: architecture.md (Phase 1, 2026-06-12)
inputDocuments:
  - _powri/planning-artifacts/prds/phase2/prd-phase2.md
  - _powri/planning-artifacts/prds/phase2/addendum.md
  - _powri/planning-artifacts/architecture/phase1/architecture-phase1.md
  - docs/analytics/tracking-plan.md
  - content/content-model.md
project_name: japan_winter_sport
user_name: Mindodoo
---

# Architecture Decision Document — Phase 2 Delta

**Powri Phase 2:** Supabase auth + UGC + maps + passport + trip skeleton.

**Phase 1 architecture remains authoritative** for foundation patterns (Next.js, content loader, analytics, CSP, rate limiting). This document adds **only Phase 2 decisions** and overrides Phase 1 "deferred" items where noted.

| Document | Role |
|----------|------|
| [`../phase1/architecture-phase1.md`](../phase1/architecture-phase1.md) | Phase 1 foundation — still read first |
| **This file** | Phase 2 additions |
| [`prds/phase2/prd-phase2.md`](../../prds/phase2/prd-phase2.md) | FR-8–17 product requirements |
| [`prds/phase2/addendum.md`](../../prds/phase2/addendum.md) | Badges, UX layout, handoff schemas |
| [`supabase/migrations/001_phase2_core.sql`](../../../../supabase/migrations/001_phase2_core.sql) | Executable schema + RLS |

---

## 1. Requirements → Architecture Mapping

| FR | Architectural components |
|----|-------------------------|
| **FR-8** Auth | Supabase Auth (Google + magic link), `@supabase/ssr`, middleware session refresh, `profiles` table |
| **FR-9** Map | Mapbox + `react-map-gl` (client), `/api/places/nearby` (curated + Google Places merge/dedup), `places_cache`, resort `latitude`/`longitude`/`curated_map_places` in content |
| **FR-10** Reviews/comments | `reviews`, `comments`, `review_photos`, Experience section client island, admin PATCH routes |
| **FR-11** Passport/badges | `passport_check_ins`, `user_badges`, `lib/badges/evaluate.ts` |
| **FR-12** Trips | `trips`, `trip_days`, `trip_activities`, quota trigger, `lib/trips/templates.ts` |
| **FR-13** AI stubs | Disabled CTAs only — no new API |
| **FR-14** SEO | SSG editorial + client UGC hydrate; aggregate rating in RSC props |
| **FR-15** Saved | `saved_resorts` + `localStorage` merge on login |
| **FR-16** Nav | `BottomNav` 4-tab, `DesktopTopNav`, `AppMenuSheet` overflow |
| **FR-17** Profile | `/u/[username]`, `/account`, `/passport` routes |

**New NFRs (PRD §6):** NFR-10–17 — sanitization, RLS, photo limits, session timeout, server-only keys, lazy map, adaptive nav.

---

## 2. Stack Additions (Phase 2)

| Layer | Phase 1 | Phase 2 addition |
|-------|---------|------------------|
| Database | None | **Supabase PostgreSQL** |
| Auth | None | **Supabase Auth** + `@supabase/ssr` |
| File storage | None | **Supabase Storage** (review + passport photos) |
| Maps | External trail map link | **Mapbox** + **react-map-gl** (dynamic import, SSR off) |
| POI data | — | **Google Places API** (server proxy) |
| UGC sanitize | Chat input only | **isomorphic-dompurify** or strip-html on all UGC |

**New npm dependencies (Epic P0):**

```bash
cd web && npm install @supabase/supabase-js @supabase/ssr mapbox-gl react-map-gl
```

---

## 3. Data Architecture

### 3.1 Dual storage model

```
┌─────────────────────────────────────────────────────────────────┐
│ Editorial (unchanged)          │ Runtime UGC (new)               │
├────────────────────────────────┼─────────────────────────────────┤
│ content/resorts/*.md           │ Supabase PostgreSQL             │
│ Build-time Zod + SSG           │ Supabase Storage (photos)       │
│ latitude/longitude [T1] NEW    │ places_cache table              │
└────────────────────────────────┴─────────────────────────────────┘
│ Guest saved slugs              │ PostHog (analytics only)        │
│ localStorage powri_saved_*     │                                 │
└────────────────────────────────┴─────────────────────────────────┘
```

### 3.2 Content schema extension

Add to `content-model.md` §4.1 and `web/src/lib/content/schema.ts`:

```yaml
latitude: 42.865087      # WGS84 [T1] — required for published resorts (Phase 2)
longitude: 140.687984
map_default_zoom: 13     # optional; default 13
```

Build warning if published resort missing coordinates; **do not fail build** until all 20 populated (then switch to hard fail).

### 3.3 Canonical SQL

Full DDL + RLS: [`supabase/migrations/001_phase2_core.sql`](../../../../supabase/migrations/001_phase2_core.sql).

**Service-role only tables:** `places_cache`, `moderation_audit` — accessed from Route Handlers with `SUPABASE_SERVICE_ROLE_KEY`, never from browser.

### 3.4 Storage buckets

| Bucket | Public read | Write |
|--------|-------------|-------|
| `review-photos` | Yes (published reviews) | Owner via signed upload URL |
| `passport-photos` | Yes | Owner via signed upload URL |

Max sizes enforced in API: 10 MB; MIME `image/jpeg`, `image/png`, `image/webp`.

---

## 4. Authentication & Session

### 4.1 Decision: Supabase SSR cookie pattern

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Client SDK | `@supabase/ssr` | Official Next.js App Router pattern; HTTP-only cookies |
| Token storage | **Cookies only** | PRD FR-8; never `localStorage` for JWT |
| Providers | Google OAuth + magic link | PRD §5.1 |
| Session refresh | Middleware `updateSession()` | Prevents stale RSC auth |
| Idle timeout | 30 days rolling | Supabase refresh token default + config |
| Absolute max | 90 days | Re-auth required; document in Privacy Policy |

### 4.2 Supabase clients (two instances)

```
web/src/lib/supabase/
├── client.ts      # createBrowserClient — client components
├── server.ts      # createServerClient — RSC, Route Handlers
├── middleware.ts  # createServerClient — session refresh
└── admin.ts       # createClient(service_role) — admin API only
```

### 4.3 Middleware update

Extend `web/src/middleware.ts`:

1. Call `updateSession(request)` for non-API routes (after `/ingest` bypass, before intl).
2. Pass `user` to RSC via cookie — do **not** trust client `is_logged_in` alone for mutations.
3. API routes: read session from cookie in each handler; return `401 UNAUTHORIZED` if required.

**Public routes (no auth):** all Phase 1 routes + `/saved` (guest local), `/u/[username]`, resort detail editorial.

### 4.4 Post-login side effects

On `auth_sign_in_completed`:

1. Alias PostHog anonymous → `user.id` (`posthog.identify`).
2. `POST /api/saved/merge` — merge `localStorage` slugs into `saved_resorts`.
3. Redirect to `returnTo` query param (magic link + OAuth callback).

### 4.5 Profile bootstrap

Trigger `handle_new_user` creates profile with temporary username `user-{idPrefix}`. App prompts username pick on first gated action if still default pattern.

**Username rules:** `[a-z0-9-]`, 3–30 chars, unique — validated in API + DB constraint.

### 4.6 Admin

- Role stored in `profiles.role`.
- Promotion: manual SQL `UPDATE profiles SET role = 'admin' WHERE id = '...'`.
- Admin routes verify role via server session + DB read (or service role for moderation).

---

## 5. API Routes (Phase 2)

**Pattern:** REST JSON under `/api/*` — extends Phase 1 error shape.

**Extended error codes** (add to `web/src/lib/api/errors.ts`):

`UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `VALIDATION_ERROR`, `TRIP_QUOTA_EXCEEDED`

| Route | Method | Auth | Purpose |
|-------|--------|------|---------|
| `/api/auth/callback` | GET | — | OAuth / magic-link callback |
| `/api/places/nearby` | GET | Public | `?resortSlug=&category=` — cached Places proxy |
| `/api/reviews` | GET | Public | `?resortSlug=` — list published |
| `/api/reviews` | POST | User | Create/update own review |
| `/api/reviews/[id]` | PATCH | User/Admin | Edit own or admin hide |
| `/api/reviews/[id]/photos` | POST | User | Upload photo (signed URL flow) |
| `/api/comments` | GET | Public | `?resortSlug=` |
| `/api/comments` | POST | User | Create comment |
| `/api/comments/[id]` | PATCH | User/Admin | Edit / hide |
| `/api/reports` | POST | User | Report review or comment |
| `/api/passport/check-ins` | GET/POST/DELETE | User | Check-in CRUD |
| `/api/passport/check-ins/[id]/photo` | POST | User | Passport photo |
| `/api/badges/evaluate` | POST | User | Re-run badge rules (also called post check-in/review) |
| `/api/trips` | GET/POST | User | List/create (max 5) |
| `/api/trips/[id]` | GET/PATCH/DELETE | User | Trip CRUD |
| `/api/trips/[id]/activities` | PATCH | User | Reorder/edit activities |
| `/api/saved` | GET/POST/DELETE | User | Synced saves |
| `/api/saved/merge` | POST | User | Merge localStorage slugs |
| `/api/account/export` | GET | User | GDPR JSON export |
| `/api/account/delete` | POST | User | Deletion request |
| `/api/admin/reviews/[id]/hide` | PATCH | Admin | Moderation |
| `/api/admin/comments/[id]/hide` | PATCH | Admin | Moderation |

**Rate limits** (extend `RATE_LIMIT_POLICIES`):

| Policy | Limit | Routes |
|--------|-------|--------|
| `auth` | 10/min/IP | magic link send, callback |
| `ugcWrite` | 20/min/user | reviews, comments, check-ins |
| `places` | 60/min/IP | `/api/places/nearby` |
| `upload` | 10/min/user | photo uploads |

### 5.1 Google Places proxy

```
GET /api/places/nearby?resortSlug=niseko-united&category=restaurant

1. Load lat/lng from build-time resort index (not client-supplied coords)
2. Check places_cache (service role) — return if fetched_at < configured TTL ago
   (env `PLACES_CACHE_TTL_HOURS`, default 168h/7d, minimum 24h)
3. Else call Google Places Nearby Search (server key) with a Pro-tier-only field
   mask — no `rating`/`userRatingCount` (Enterprise SKU trigger; see addendum §F)
4. Upsert cache; return minimal fields: placeId, name, lat, lng, vicinity
5. Response: { places: [...], attribution: "Powered by Google" }
```

**CSP update:** allow Mapbox hosts (`api.mapbox.com`, `*.tiles.mapbox.com`, `events.mapbox.com`) and `worker-src blob:` in `lib/security/csp.ts`.

---

## 6. Frontend Architecture (Phase 2)

### 6.1 New routes (`web/src/app/[locale]/`)

| Route | Rendering | Notes |
|-------|-----------|-------|
| `saved/page.tsx` | RSC + client hearts | Guest reads localStorage client-side |
| `plan/page.tsx` | RSC shell + client | Empty state if logged out |
| `plan/[tripId]/page.tsx` | Client-heavy | Timeline editor |
| `passport/page.tsx` | RSC + client | Owner view |
| `u/[username]/page.tsx` | RSC | Public profile; `noindex` meta |
| `account/page.tsx` | Client | Settings, export, delete |

Resort detail: add `<ExperienceSection />`, `<ResortMapSection />`, `<SaveResortButton />` to existing `ResortDetailContent`.

### 6.2 Component folders

```
web/src/components/
├── auth/           # SignInSheet, AuthProvider
├── experience/     # ReviewList, ReviewForm, CommentThread, AggregateRating
├── map/            # ResortMapSection, PlacePinPopup (dynamic, ssr: false)
├── passport/       # PassportGrid, CheckInButton, BadgeToast
├── plan/           # TripList, TripTimeline, ActivityRow
├── profile/        # PublicProfile, ProfileBadgeShelf
├── saved/          # SaveResortButton, SavedList
└── layout/
    ├── BottomNav.tsx      # 4 items: Home Explore Saved Plan
    ├── DesktopTopNav.tsx  # md+ breakpoint
    └── AppMenuSheet.tsx   # Passport Info Account overflow
```

### 6.3 New lib modules

```
web/src/lib/
├── supabase/       # clients (§4.2)
├── badges/
│   ├── definitions.ts   # 16 badges from addendum §C
│   ├── evaluate.ts      # idempotent award logic
│   └── regions.ts       # slug → badge region from content index
├── trips/
│   ├── templates.ts     # generate itinerary from date range
│   └── types.ts
├── places/
│   ├── nearby.ts        # server fetch + cache
│   └── categories.ts
├── ugc/
│   ├── sanitize.ts      # strip HTML, length limits
│   └── profanity.ts     # display name blocklist
├── saved/
│   ├── localStorage.ts  # guest saves
│   └── merge.ts
└── analytics/
    └── phase2Events.ts  # registry mirroring phase1Events.ts
```

### 6.4 Adaptive navigation (FR-16)

| Viewport | Nav |
|----------|-----|
| `< 768px` | Bottom: Home · Explore · Saved · Plan. Hamburger: Passport · Info · Account |
| `≥ 768px` | Top nav all items; bottom nav `hidden md:hidden` |

Implement `DesktopTopNav` in `AppShell`; use Tailwind `md:` breakpoint consistently.

### 6.5 Map loading

```tsx
// dynamic import — Mapbox GL JS requires window
const ResortMapSection = dynamic(
  () => import('@/components/map/ResortMapSection'),
  { ssr: false, loading: () => <MapSkeleton /> }
);
```

Intersect observer fires `map_section_viewed` once per page load.

### 6.6 SEO hybrid (FR-14.3)

| Content | Render strategy |
|---------|-----------------|
| Editorial sections | SSG/RSC — unchanged |
| Aggregate rating + count | RSC fetch from Supabase (cached `revalidate: 60`) |
| Review list + comments | Client fetch `/api/reviews?resortSlug=` after hydration |
| Profile pages | SSR with `robots: noindex, follow` |

---

## 7. Badge Evaluation

**Location:** `lib/badges/evaluate.ts` — called after check-in insert and review publish.

**Region lookup:** join `resort_slug` → build-time `Resort.region` (Hokkaido/Nagano/Tohoku/Niigata) per addendum §C.1.

**Season:** JST Nov 1 – Apr 30 for `mountain_hopper`, `season_starter`, `spring_rider`.

**Idempotent:** insert into `user_badges` on conflict do nothing; fire `badge_earned` only on new rows.

---

## 8. Trip Template Generation

`lib/trips/templates.ts`:

```typescript
generateTripTemplate(resortSlug: string, startDate: Date, endDate: Date): TripPayload
```

- Day count = `differenceInDays(end, start) + 1`
- Day 1: airport → transport → accommodation → restaurant
- Middle days: ski → restaurant (optional attraction)
- Last day: ski (optional) → transport → airport
- Set `ai_ready: true` when resort + dates present

Quota enforced in DB trigger + API pre-check (friendly error `TRIP_QUOTA_EXCEEDED`).

---

## 9. Analytics (Phase 2)

**Authority:** update `docs/analytics/tracking-plan.md` §3 before first instrumentation PR.

**Registry:** `web/src/lib/analytics/phase2Events.ts`  
**Test script:** extend `scripts/verify-analytics-events.ts` or add `test:analytics:phase2`

**Identity:** set `is_logged_in: true` in `getCommonProperties()` when Supabase session exists.

**Alias:** on sign-in complete, `posthog.alias(userId)`.

Canonical events — see PRD §10.4 (supersedes old tracking-plan `accommodation_map_interacted`, `trip_saved`).

---

## 10. Environment Variables

Add to `.env.example` and Vercel:

```bash
# Supabase (Phase 2)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=          # server only — never NEXT_PUBLIC

# Google Places (Phase 2)
GOOGLE_PLACES_API_KEY=              # server only

# Existing Phase 1 vars unchanged
```

**Supabase Auth redirect URLs:**  
`https://<domain>/api/auth/callback` (+ localhost for dev)

---

## 11. Security Checklist (Phase 2)

| Control | Implementation |
|---------|----------------|
| RLS on all user tables | Migration 001 |
| Service role server-only | `lib/supabase/admin.ts` — never import in client |
| UGC sanitization | `lib/ugc/sanitize.ts` on every write |
| Photo MIME verify | Magic bytes check server-side |
| CSRF | Same-origin + cookie auth (no custom CSRF token needed for JSON API) |
| Admin routes | Session + `role === admin'` or service role |
| CSP | Add Mapbox tile/style hosts + `worker-src blob:`; Google attribution in UI not script |
| Rate limits | Extended policies §5 |

---

## 12. Project Structure Delta

```
japan_winter_sport/
├── supabase/
│   └── migrations/
│       └── 001_phase2_core.sql          # NEW
├── content/resorts/*.md                 # + latitude, longitude
└── web/src/
    ├── app/[locale]/
    │   ├── saved/
    │   ├── plan/[tripId]/
    │   ├── passport/
    │   ├── account/
    │   └── u/[username]/
    ├── app/api/
    │   ├── auth/callback/
    │   ├── places/nearby/
    │   ├── reviews/ ...
    │   ├── trips/ ...
    │   └── admin/ ...
    ├── components/{auth,experience,map,passport,plan,profile,saved}/
    └── lib/{supabase,badges,trips,places,ugc,saved}/
```

---

## 13. Implementation Sequence (P0 → P1)

| Step | Deliverable | FR |
|------|-------------|-----|
| 1 | Supabase project + migration 001 + env vars | P0 |
| 2 | `lib/supabase/*` + middleware session + auth callback | FR-8 |
| 3 | SignInSheet + account routes + profile bootstrap | FR-8, FR-17 |
| 4 | Adaptive nav (BottomNav + DesktopTopNav + menu) | FR-16 |
| 5 | Saved resorts localStorage + API + `/saved` | FR-15 |
| 6 | Content lat/lng + curated places schema + Places proxy (merge/dedup) + map UI | FR-9 |
| 7 | Reviews + comments + Experience section | FR-10 |
| 8 | Passport + badges + `/passport` + public profile | FR-11, FR-17 |
| 9 | Trips CRUD + template + Plan UI | FR-12 |
| 10 | AI stubs + phase2Events + tracking-plan sync | FR-13 |
| 11 | Admin hide routes + moderation audit | FR-10.4 |
| 12 | GDPR export/delete + legal pages | FR-8.4 |

**Do not start step 6 until lat/lng populated for target resorts.**  
**Do not start step 7 until legal/community guidelines drafted (readiness §B5–6).**

---

## 14. Phase 1 Architecture Overrides

| Phase 1 statement | Phase 2 override |
|-------------------|-------------------|
| "No DB Phase 1" | Supabase PostgreSQL from Phase 2 |
| "Deferred: Supabase auth" | **Implemented** |
| "Deferred: Google Places" | **Implemented** |
| "Deferred: Leaflet map" | **Implemented** (surroundings map — Mapbox + react-map-gl) |
| "Deferred: Plan tab" | **Trip skeleton** (AI fill still Phase 3) |
| Reviews in phase-3 epic | **Phase 2** per PRD |

---

## 15. Validation & Readiness

| Check | Status |
|-------|--------|
| All FR-8–17 mapped to components/routes | ✅ |
| SQL migration + RLS documented | ✅ |
| API contract listed | ✅ |
| Env vars listed | ✅ |
| Phase 1 patterns preserved (errors, analytics, CSP) | ✅ |
| UX wireframes | ⏳ `bmad-ux` — addendum §M is layout spec until UX pass |
| Tracking plan §3 synced | ⏳ before first analytics PR |
| CONTENT_MODEL lat/lng | ⏳ content task |

**Architecture readiness:** **READY for P0 (auth + Supabase setup)** after Supabase project provisioned.

---

*Phase 2 architecture delta complete — 2026-06-23. Next: `bmad-create-epics-and-stories` · `bmad-ux` · sync tracking-plan §3.*
