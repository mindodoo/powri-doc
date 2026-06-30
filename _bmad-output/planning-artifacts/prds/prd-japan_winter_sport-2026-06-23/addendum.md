# Powri Phase 2 PRD — Addendum

**Companion to:** [`prd.md`](prd.md)  
**Purpose:** Technical depth, badge catalog, schemas, and implementation notes that belong outside the main PRD narrative.

---

## A. Authentication Setup Notes

### Google OAuth (recommended — ship first)

| Aspect | Detail |
|--------|--------|
| Effort | ~0.5–1 day with Supabase Auth |
| Steps | Google Cloud Console OAuth client → Supabase Auth provider → redirect URLs for Vercel |
| AI agent fit | Well-documented; Supabase + Next.js patterns are agent-friendly |

### Apple Sign-In (not Phase 2)

| Aspect | Detail |
|--------|--------|
| Effort | ~2–4 days first time; ongoing Apple Developer Program ($99/yr) |
| Extra work | Services ID, domain verification, JWT key rotation, App Store guidelines for web |
| PO decision | Deferred — Google + email covers majority of international ski audience |

### Email auth — magic link (confirmed Phase 2)

| Option | Pros | Cons |
|--------|------|------|
| **Magic link** ✅ | No password to forget; lower friction; no password breach surface | Some users unfamiliar; email deliverability setup (SPF/DKIM) |
| Email + password | Familiar to older web users | Password reset flows, breach risk, weaker UX |

**Recommendation:** Supabase magic link + Google OAuth. Session via HTTP-only cookies; PostHog alias on first login.

### A.1 Magic-link trust UX (addressing "feels less safe")

Magic links are **more secure** than passwords for most users (no reusable credential, no password reuse across sites, short-lived token). The product risk is **perceived** safety, not actual safety. Mitigate with UX:

| Moment | Pattern | Example copy |
|--------|---------|--------------|
| **Sign-in sheet** | Lead with Google; email as second option | "Continue with Google" (primary button) · "Or email me a secure link" |
| **Email input** | Explain *why* before asking for email | "We'll send a one-time sign-in link. No password to create or remember." |
| **Trust signals** | Icon + expiry + privacy | 🔒 "Link expires in 15 minutes · Works once · We never see your password" |
| **After submit** | Confirmation state, not spinner only | "Check your inbox — we sent a link to m•••@gmail.com" + "Open Gmail" deep link where possible |
| **Anxiety reducer** | Spam folder | "Nothing there? Check spam, or try Google sign-in instead." |
| **Link landing** | Return to intent | Land on same resort/trip/review flow — not a generic homepage |
| **Privacy policy** | One line link | "How we handle your email" → Privacy § auth |

**Do not:**
- Use scary security jargon ("token", "JWT") in user-facing copy.
- Hide Google behind email — users who distrust magic links often trust Google.
- Send magic link without clear sender name (`Powri` or `Powri <noreply@...>`).

**Measure:** `magic_link_sent` → `magic_link_completed` conversion (target SM-9: ≥30% within 15 min). If below 20% after launch, A/B test copy and Google button prominence.

**Optional Phase 2.1 (only if data shows problem):** Add "Prefer a password?" — defer unless magic-link completion is poor; passwords add scope you explicitly avoided.

### Session persistence ("lock-in")

- Same device: refresh token in secure cookie — user stays signed in across browser restarts until idle timeout.
- Same IP is **not** a reliable auth signal (mobile networks, VPNs) — use cookie/session only, not IP binding.
- Document in Privacy Policy: session stored on device; user can sign out everywhere from account settings `[Phase 3 if multi-device revoke needed]`.

---

## B. Data Storage Architecture

**PostHog is not sufficient for UGC.** PostHog stores analytics events only — not reviews, photos, trips, or passport data.

### Recommended stack

```
┌─────────────────────────────────────────────────────────┐
│  Next.js (Vercel)                                       │
│  ├── SSG resort pages (markdown — unchanged)              │
│  ├── API routes (/api/reviews, /api/trips, /api/places) │
│  └── Server Components + client islands for UGC           │
└─────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌──────────────┐    ┌─────────────────┐   ┌──────────────┐
│  Supabase    │    │  Supabase       │   │  PostHog     │
│  PostgreSQL  │    │  Storage        │   │  (analytics) │
│  (UGC, trips)│    │  (photos)       │   │              │
└──────────────┘    └─────────────────┘   └──────────────┘
```

### PostgreSQL tables (conceptual)

| Table | Key fields |
|-------|------------|
| `profiles` | `id` (FK auth.users), `display_name`, `avatar_url`, `bio`, `role` (`user`/`admin`) |
| `reviews` | `id`, `user_id`, `resort_slug`, `rating`, `body`, `status` (`published`/`hidden`/`deleted`), timestamps |
| `review_photos` | `id`, `review_id`, `storage_path`, `sort_order` |
| `comments` | `id`, `user_id`, `resort_slug`, `body`, `status`, timestamps |
| `passport_check_ins` | `id`, `user_id`, `resort_slug`, `visited_at`, `note`, `photo_path` nullable |
| `user_badges` | `user_id`, `badge_id`, `earned_at` (unique pair) |
| `trips` | see § Trip Object Schema |
| `trip_days` | `id`, `trip_id`, `day_number`, `date` |
| `trip_activities` | `id`, `trip_day_id`, `type`, `title`, `notes`, `sort_order` |
| `content_reports` | `id`, `reporter_id`, `content_type`, `content_id`, `reason`, `status` |
| `moderation_audit` | `id`, `admin_id`, `action`, `content_type`, `content_id`, timestamp |
| `saved_resorts` | `user_id`, `resort_slug`, `saved_at` (unique pair) |

**Guest saved resorts:** `localStorage` key `powri_saved_resorts` (array of slugs). On login, API merges into `saved_resorts` and clears local key.

### Supabase Storage buckets

| Bucket | Path pattern | Access |
|--------|--------------|--------|
| `review-photos` | `{user_id}/{review_id}/{uuid}.webp` | Public read if review published; write owner only |
| `passport-photos` | `{user_id}/{check_in_id}/{uuid}.webp` | Public read; write owner only |

### Photo processing pipeline

1. Client validates: ≤5 photos/review, ≤1/check-in, ≤10 MB each, JPEG/PNG/WebP.
2. Server validates MIME + dimensions; reject executables.
3. Optional: resize to max 2048px long edge, convert to WebP (~80% quality) before storage.
4. Store URL in DB; serve via Supabase CDN or signed URLs.

### Estimated storage at launch scale

| Assumption | Volume |
|------------|--------|
| Users | 500 |
| Reviews with photos | 200 × 2 photos avg × 500 KB = ~200 MB |
| Passport photos | 300 × 500 KB = ~150 MB |
| **Total** | < 1 GB — well within Supabase free tier |

---

## C. Badge Catalog & Award Rules

Badges auto-award on check-in or review milestones. Each badge has: `id`, `name`, `description`, `icon_key`, `tier`, `condition`.

### Tier system

| Tier | Visual treatment |
|------|------------------|
| **Bronze** | Warm copper accent ring, single star motif |
| **Silver** | Cool silver ring, double line border |
| **Gold** | Soft gold gradient ring, subtle glow |
| **Regional** | Prefecture/region silhouette watermark |
| **Special** | Seasonal or limited — animated shimmer on unlock only |

### Badge UI design rules (for AI asset generation)

Align with Powri Theme A ([`prd/12-ux-theme-and-layout.md`](../../prd/12-ux-theme-and-layout.md)):

| Token | Badge application |
|-------|-------------------|
| Canvas | 96×96 px display (export @2x 192×192); circular crop |
| Background | Warm off-white `#FAF8F5` or soft gradient to icy blue `#E8F4F8` |
| Border | 2px ring — tier color; no harsh black strokes |
| Icon style | Flat vector + subtle texture; single focal motif; no photorealism |
| Typography | Badge name **not** inside icon — name renders in DM Sans below grid |
| Corner radius | N/A (circle) |
| Shadow | `0 2px 8px rgba(0,0,0,0.08)` on grid tile only |
| Locked state | 40% opacity + dashed ring; no negative/dark imagery |
| Grid layout | 3 columns mobile, 4–5 desktop; 16px gap |

**Prompt template for AI badge generation:**

> Flat circular badge icon for a ski travel app. Warm off-white background, soft icy blue accent, minimal flat vector illustration of [MOTIF], premium travel magazine aesthetic, no text, no photorealism, 1:1, consistent line weight with previous badges.

### Badge definitions

| ID | Name | Tier | Condition |
|----|------|------|-----------|
| `first_tracks` | First Tracks | Bronze | 1 resort check-in |
| `double_drop` | Double Drop | Bronze | 2 resort check-ins |
| `mountain_hopper` | Mountain Hopper | Silver | 3–5 resort check-ins (same season `[ASSUMPTION: Nov–Apr]`) |
| `powder_hunter` | Powder Hunter | Gold | 5 resort check-ins (lifetime) |
| `japan_ski_veteran` | Japan Ski Veteran | Gold | 10 resort check-ins (lifetime) |
| `hokkaido_explorer` | Hokkaido Explorer | Regional | 3 Hokkaido resorts checked in |
| `nagano_nomad` | Nagano Nomad | Regional | 3 Nagano prefecture resorts |
| `tohoku_trailblazer` | Tohoku Trailblazer | Regional | 2 Tohoku region resorts |
| `weekend_warrior` | Weekend Warrior | Bronze | 2 check-ins within same 7-day window |
| `season_starter` | Season Starter | Special | First check-in before Dec 15 |
| `spring_rider` | Spring Rider | Special | Check-in at resort in April–May |
| `super_reviewer` | Super Reviewer | Silver | 3 published reviews |
| `master_mind` | Master Mind | Gold | 10 published reviews |
| `ski_god` | Ski God | Gold | 20 published reviews **OR** all 20 seed resorts checked in |
| `photo_memoir` | Photo Memoir | Bronze | 1 passport photo uploaded |
| `storyteller` | Storyteller | Silver | 5 reviews with ≥1 photo each |

**Evaluation:** Run badge rules in DB trigger or API after check-in/review insert; idempotent — never revoke badges.

### C.1 Badge region mapping (all 20 seed resorts)

Regional badges use the seed content `region` field. Canonical mapping for Phase 2:

| `badge_region` | Count | Resort slugs |
|----------------|-------|--------------|
| **hokkaido** | 6 | `furano`, `niseko-united`, `rusutsu`, `sahoro`, `kiroro`, `tomamu` |
| **nagano** | 5 | `hakuba-valley`, `shiga-kogen`, `nozawa-onsen`, `madarao-kogen`, `tsugaike-kogen` |
| **tohoku** | 3 | `appi-kogen`, `zao-onsen`, `nekoma-alts-bandai` |
| **niigata** | 6 | `gala-yuzawa`, `joetsu-kokusai`, `kandatsu-kogen`, `lotte-arai`, `myoko-kogen`, `naeba` |

**Badge rules using regions:**
- `hokkaido_explorer` — 3 check-ins where `region = Hokkaido`
- `nagano_nomad` — 3 check-ins where `region = Nagano`
- `tohoku_trailblazer` — 2 check-ins where `region = Tohoku`

Niigata resorts count toward lifetime totals (`powder_hunter`, etc.) but have no regional badge in Phase 2.

Implementation: join `passport_check_ins.resort_slug` → build-time resort index → `region` enum. Optional: add `badge_region` to frontmatter if `region` string ever diverges.

### C.2 Ski season boundaries (badge logic)

| Rule | Value |
|------|-------|
| Season window | **1 November – 30 April** (inclusive), **Asia/Tokyo (JST)** |
| `mountain_hopper` | 3–5 distinct resort check-ins with `visited_at` falling in the **same** season window |
| `season_starter` | First check-in of season with `visited_at` before **15 Dec** |
| `spring_rider` | Check-in with `visited_at` in **April or May** |

---

## D. Trip Object Schema

### JSON shape (API + Phase 3 AI contract)

```typescript
interface Trip {
  id: string;                    // uuid
  user_id: string;
  title: string;
  status: 'draft' | 'planned' | 'completed';
  resort_slug: string;           // primary resort
  start_date: string;            // ISO date
  end_date: string;
  ai_ready: boolean;             // true when resort + dates set
  ai_populated_at: string | null; // Phase 3
  created_at: string;
  updated_at: string;
  itinerary_days: TripDay[];
}

interface TripDay {
  id: string;
  day_number: number;            // 1-based
  date: string;                  // ISO date
  activities: TripActivity[];
}

interface TripActivity {
  id: string;
  type:
    | 'airport'
    | 'transport'
    | 'accommodation'
    | 'ski'
    | 'restaurant'
    | 'attraction'
    | 'custom';
  title: string;
  notes: string | null;
  sort_order: number;
  // Phase 3 extensions (nullable in Phase 2):
  place_id?: string | null;      // Google Places ID
  seed_content_ref?: string | null; // e.g. "getting_there.shuttle"
  ai_generated?: boolean;
}
```

### Default template (3-night trip example)

| Day | Activities (in order) |
|-----|----------------------|
| 1 | `airport` Airport arrival → `transport` Transfer to resort → `accommodation` Check in → `restaurant` Dinner nearby |
| 2 | `ski` Full day on mountain → `restaurant` Après / dinner |
| 3 | `ski` Full day on mountain → `attraction` Onsen or village stroll → `restaurant` Dinner |
| 4 | `ski` Half day (optional) → `transport` Transfer to airport → `airport` Departure |

Template length = `(end_date - start_date) + 1` days; adjust if single-day trip edge case.

---

## E. Phase 3 AI Handoff Requirements

Phase 2 must deliver these **contracts** so Phase 3 AI can implement without schema migration:

### E.1 Entry points (UI)

| Surface | CTA copy | Phase 2 behaviour | Phase 3 behaviour |
|---------|----------|-------------------|-------------------|
| Resort detail | "Plan with AI" | Stub modal → sign-in / create trip | Open chat with `resort_slug` context |
| Quiz results | "Want help planning?" | Stub → top match resort pre-selected for trip | Chat with quiz answers in context payload |
| Trip detail | "Fill with AI" | Disabled button + tooltip | POST `/api/trip/fill` → populate activities |

### E.2 Context payload for AI (Phase 3)

Phase 2 stores trip shell; Phase 3 AI request includes:

```json
{
  "trip_id": "uuid",
  "resort_slug": "niseko-united",
  "start_date": "2026-02-10",
  "end_date": "2026-02-14",
  "itinerary_days": [ "...existing skeleton..." ],
  "user_context": {
    "locale": "en",
    "passport_resorts": ["niseko-united", "hakuba-happo"],
    "saved_resorts": ["niseko-united", "nozawa-onsen"],
    "review_count": 2
  },
  "quiz_context": null
}
```

`quiz_context` is **null in Phase 2** when quiz results aren't persisted. Phase 3 options:
- Pass quiz answers in-session only (query param / sessionStorage handoff from quiz → trip).
- Or add optional "Save my quiz preferences" in Phase 3.

### E.3 AI fill rules (Phase 3 — document now)

- AI **replaces placeholder titles/notes** on existing activities — does not delete user-edited activities marked `user_locked` `[Phase 3 field]`.
- AI pulls transport/accommodation suggestions from **seed content first** ([`prd/08-ai-content.md`](../../prd/08-ai-content.md)).
- Google Places IDs may attach to `restaurant` / `attraction` activities — Phase 2 schema leaves fields nullable.
- Cost guardrails from Phase 3 PRD apply (10 turns/session, etc.).

### E.4 Quiz ↔ Profile separation (PO decision)

| Data | Phase 2 | Phase 3 |
|------|---------|---------|
| Quiz answers | Not stored on profile | Optional in-session handoff to AI |
| Quiz results link | CTA stub only | AI entry with live context |
| Trip | Stored on profile | AI-enriched |

---

## F. Google Places API — Cost & Implementation

| Control | Implementation |
|---------|----------------|
| Proxy | `/api/places/nearby?resort_slug=&category=` — server-side key |
| Cache | Redis or Supabase table; TTL 24h per resort+category |
| Fields | Minimal: name, place_id, lat/lng, rating, vicinity |
| Fallback | Static map pin + "Open in Google Maps" search link |

At 20 resorts and early traffic, expect **< $50/month** — within $200 Google credit.

---

## G. Moderation Without Admin UI

Phase 2 admin workflow:

1. User reports → row in `content_reports`.
2. Admin receives email digest `[ASSUMPTION: daily]` or checks Supabase dashboard.
3. Admin runs SQL or calls `PATCH /api/admin/reviews/:id/hide` with service role key.
4. Phase 3: proper admin UI with queue.

---

## H. GDPR / PDPA Account Lifecycle

| Action | Behaviour |
|--------|-----------|
| Export | JSON of profile, reviews, comments, trips, check-ins, badge list |
| Delete request | Immediate sign-out; 30-day grace; hard delete/anonymize |
| Review on delete | Anonymize author to "Deleted user"; keep text for community integrity `[ASSUMPTION — confirm legal]` |
| Photos | Deleted from storage on account deletion |

---

## I. Reconciliation with Phase 1 Roadmap Docs

| Phase 1 doc said | Phase 2 PRD decision |
|------------------|---------------------|
| Reviews in Phase 3 community epic | **Moved to Phase 2** per PO — auth is Phase 2 anyway |
| Supabase Phase 3 | **Moved to Phase 2** — required for trips, reviews, passport |
| Plan tab Phase 3 | **Skeleton Phase 2**; AI fill Phase 3 |
| FR-6 AI chat Phase 3 | Unchanged — stubs only Phase 2 |

Update [`prd/11-roadmap-scope.md`](../../prd/11-roadmap-scope.md) and phase-3 epics when Phase 2 PRD is finalized.

---

## J. Navigation — Phase 2 IA (confirmed)

Powri ships as **mobile-first web** (not a native app). Phase 1 already has bottom nav (3 items) + hamburger duplicate — Phase 2 refactors rather than replaces.

### Mobile (<768px)

```
┌─────────────────────────────────────┐
│  [☰]  Powri / Search          [👤]  │  ← top bar (hamburger + account)
├─────────────────────────────────────┤
│                                     │
│            page content             │
│                                     │
├─────────────────────────────────────┤
│  Home   Explore   Saved   Plan      │  ← bottom nav (4 items)
└─────────────────────────────────────┘

Hamburger overflow: Passport · Info · Sign in/out
```

### Desktop (≥768px)

```
┌──────────────────────────────────────────────────────────┐
│  Powri   Home  Explore  Saved  Plan  Passport  Info  [👤] │  ← top nav only
├──────────────────────────────────────────────────────────┤
│                      page content                         │
└──────────────────────────────────────────────────────────┘
(no bottom nav)
```

### Why not hamburger-only?

| Factor | Bottom nav (mobile) | Hamburger-only |
|--------|---------------------|----------------|
| Thumb reach on 375px | ✅ | ❌ extra tap |
| Discoverability of Plan/Saved | ✅ always visible | ❌ buried |
| Desktop web norms | Hide it | ✅ natural |
| Phase 2 item count | 4 fits | OK for overflow |

### Implementation note

Extend existing `BottomNav.tsx` and `AppMenuSheet.tsx` — remove duplicate Home/Explore/Info from hamburger on mobile; hamburger becomes **overflow only**.

---

## K. Content Prerequisites — Resort Geo Fields

Map FR-9 requires coordinates in seed content. Add to `CONTENT_MODEL.md` §4.1:

```yaml
latitude: 42.1234          # WGS84 decimal degrees [T1]
longitude: 140.5678        # WGS84 decimal degrees [T1]
map_default_zoom: 13       # optional; default 13 if omitted [T1]
```

| Requirement | Detail |
|-------------|--------|
| Source | Official resort base area / main gondola — one pin per slug |
| Validation | Zod: lat −90..90, lng −180..180; build warns if missing on published resort |
| Blocker | Map story cannot ship until all **20** published resorts populated |
| Task owner | Editorial / one-time GIS research (~2–4 hrs total) |

---

## L. Map Implementation — Leaflet + Google Places

| Layer | Technology |
|-------|------------|
| Map tiles | **Leaflet** + **OpenStreetMap** (Phase 1 architecture default) |
| Resort pin | Seed content lat/lng |
| Nearby POIs | **Google Places API** via `/api/places/nearby` server proxy |
| Cache | Supabase table or edge cache; TTL 24h per `resort_slug + category` |

**Google Places attribution (required on map UI):**
- Display "Powered by Google" or equivalent per [Places API policies](https://developers.google.com/maps/documentation/places/web-service/policies).
- Place details shown: name, rating, vicinity only — no caching beyond TTL.

**Analytics canonical names** (supersedes old tracking-plan §3 `poi_type` / `accommodation_map_interacted`):
- `map_section_viewed`
- `map_pin_tapped` with `place_category` (not `poi_type`)

---

## M. Experience Section — Resort Detail Layout

Order of sections on `/resort/[slug]` after Phase 2:

| # | Section | Auth | Render |
|---|---------|------|--------|
| 1 | Hero + overview | Public | SSG |
| 2 | Highlights / lowlights | Public | SSG |
| 3 | Terrain / trail map link | Public | SSG |
| 4 | Getting There | Public | SSG |
| 5 | **Around the resort** (map) | Public | Lazy client + SSR shell |
| 6 | **How people experienced it** | Read public; write gated | Aggregate SSR; list hydrated |
| 7 | Practical tips (if present) | Public | SSG |

**"How people experienced it" internal layout:**
1. Section header + aggregate ★ rating + count
2. "Write a review" button (→ sign-in if guest)
3. Review cards (author → links to `/u/:username`, badges chip, photos)
4. Subheader: "Quick thoughts" or "Comments"
5. Flat comment list (newest first, max 50 visible + "Load more")
6. "Add a comment" (→ sign-in if guest)

---

## N. Public Profile & Account Routes

| Route | Access | Contents |
|-------|--------|----------|
| `/u/:username` | Public read | Avatar, name, bio, badge grid, passport resorts, review excerpts |
| `/passport` | Logged-in owner | Full passport + edit check-ins |
| `/account` | Logged-in owner | Edit profile, export JSON, delete account, sign out |
| `/plan` | Logged-in for CRUD; public empty state | Trip list + create |
| `/saved` | Public | Saved resort cards |

**Username rules:** lowercase slug from display name; `[a-z0-9-]`; 3–30 chars; unique constraint in DB.

---

## O. Session, Admin & Profanity

### Session (confirmed)

| Setting | Value |
|---------|-------|
| Rolling idle timeout | 30 days |
| Absolute max session | 90 days |
| Storage | HTTP-only secure cookie via Supabase SSR helper |

### Admin assignment

1. Create user via normal auth.
2. PO runs SQL: `UPDATE profiles SET role = 'admin' WHERE id = '<uuid>';`
3. Admin API routes check `role` on every request.

### Display name profanity filter

- Server-side blocklist (~200 terms EN + common variants); reject on save with friendly error.
- No ML moderation Phase 2 — UGC text moderated via report queue.

---

## P. Tracking Plan Alignment

When implementing, replace `docs/tracking-plan.md` §3 with PRD §10.4 events. Deprecated / renamed:

| Old (tracking-plan §3) | New (PRD canonical) |
|------------------------|---------------------|
| `accommodation_map_interacted` | `map_section_viewed` |
| `map_pin_tapped` + `poi_type`, `poi_name` | `map_pin_tapped` + `place_category` |
| `trip_saved` (Phase 3) | `trip_created` (Phase 2) |
| `account_created` | `auth_sign_in_completed` with `is_new_user: true` |

Add `phase2Events.ts` registry mirroring `phase1Events.ts` when first story ships.
