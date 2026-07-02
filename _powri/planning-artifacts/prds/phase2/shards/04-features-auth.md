<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.1: Authentication & Account Management (FR-8)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.1 · **See also:** **Epic:** [`epic-09-supabase-auth.md`](../../../epics/phase2/shards/epic-09-supabase-auth.md) · **Addendum:** [`addendum.md`](../addendum.md) §A · [`INDEX.md`](../../../INDEX.md)

---

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

**User** has display name, profile photo (from Google OAuth or uploaded photo), and optional short bio.

**Consequences:**
- Display name shown on reviews and comments (public identity — no anonymous mode).
- User can edit display name; server rejects display names matching the profanity blocklist (addendum §O).
- On `/account`, user uploads a photo to Supabase Storage bucket `avatars` (migration `003_avatars_bucket.sql`). Without a photo, UI shows display name initial.
- Upload constraints: JPG, PNG, or WebP; max 2 MB; validated client-side before upload and server-side on `PATCH avatar_url`.
- Profile photo auto-saves to `profiles.avatar_url`, updates the preview immediately, and reloads on later visits (cache-bust query on upload URLs).

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
