---
baseline_commit: 20d10e12b2d0e3955839dbb729737744dec359c8
---

# Story 9.5: GDPR Data Export & Account Deletion

Status: review

<!-- Validation optional: bmad-create-story before bmad-dev-story -->

## Story

As a **signed-in user**,
I want to export my data or delete my account,
So that I control my personal information under GDPR/PDPA.

## Acceptance Criteria

1. **Given** I am signed in on `/account`  
   **When** I tap **Export my data**  
   **Then** the browser downloads a JSON file containing: profile, reviews, comments, trips (with nested days and activities), passport check-ins, badges, saved resorts  
   **And** empty categories are included as `[]` when the user has no rows yet

2. **When** I request account deletion  
   **Then** a confirm step explains the 30-day policy and requires explicit confirmation (not a single accidental tap)  
   **And** on confirm: `deletion_requested_at` is recorded, I am signed out immediately, and I cannot sign back in while deletion is pending  
   **And** `account_deletion_requested` fires before session revocation

3. **And** `/account` destructive section follows UX spec: separated from profile form by section gap — **Export my data** · **Delete account** · **Sign out** (Sign out remains last)

4. **And** Privacy Policy copy updates are **deferred to Story 15.3** — add only a short in-app note linking to existing `/privacy` where helpful; do not rewrite legal pages in this story

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [x] **Unit (Vitest):** export payload assembler (shape + empty arrays); deletion guard (already pending, unauthenticated)
- [x] **Analytics:** `account_deletion_requested` wired in `phase2Events.ts` + `docs/analytics/tracking-plan.md` §3.1 instrumentation map; `npm run test:analytics`
- [x] **Schema:** `npm run verify:supabase` includes `profiles.deletion_requested_at` after migration 004
- [x] **PR gates:** `lint`, `build`, `test:unit`, `test:launch`, `test:analytics`
- [ ] **Manual E2E:** export downloads valid JSON on `/account`; delete → confirm → redirected signed out; re-login blocked with friendly message
- [ ] **Manual E2E:** user with only profile (no UGC/trips yet) still gets complete export structure

## Tasks / Subtasks

- [x] **Migration 004** (AC: 2)
  - [x] `supabase/migrations/004_account_deletion.sql` — add `deletion_requested_at timestamptz null` to `profiles`
  - [x] RLS: block `profiles` self-update when `deletion_requested_at is not null` (owner cannot undo deletion request)
  - [x] Update `scripts/verify-supabase-schema.mjs` to assert column exists

- [x] **`GET /api/account/export`** (AC: 1)
  - [x] Auth via `requireUser()`; reject if `deletion_requested_at` set (`FORBIDDEN`)
  - [x] Query user-owned rows from: `profiles`, `reviews`, `comments`, `trips` + `trip_days` + `trip_activities`, `passport_check_ins`, `user_badges`, `saved_resorts`
  - [x] Return `{ exported_at, profile, reviews, comments, trips, passport_check_ins, badges, saved_resorts }` with `Content-Disposition: attachment; filename="powri-data-export-YYYY-MM-DD.json"`
  - [x] Extract assembly logic to `web/src/lib/auth/accountExport.ts` for unit tests

- [x] **`POST /api/account/delete`** (AC: 2)
  - [x] Auth required; idempotent guard if already pending (`CONFLICT` or success-no-op — pick one and test)
  - [x] Set `deletion_requested_at = now()` (service role or allowed RLS update)
  - [x] Fire `account_deletion_requested` via `track()` **before** sign-out
  - [x] Global sign-out: `createAdminClient().auth.admin.signOut(userId, 'global')`
  - [x] Ban re-login until purge: `auth.admin.updateUserById(id, { ban_duration: '720h' })` (30 days) or equivalent documented approach
  - [x] Return `{ ok: true }`; client redirects to `/` after cookies clear

- [x] **Re-login block** (AC: 2)
  - [x] In `/api/auth/callback` (and magic-link completion path if applicable): if `profiles.deletion_requested_at` is set → sign out, redirect with `?auth=deletion_pending` (or similar) and do not establish session
  - [x] `AuthProvider` / sign-in sheet: surface friendly copy when deletion pending

- [x] **UI — `/account` destructive section** (AC: 1, 3)
  - [x] Extend `AccountSettingsForm` (or sibling `AccountDangerZone.tsx`) below profile Save button, separated by `gap-section-gap` / border-top per UX
  - [x] **Export my data** — calls export route, triggers blob download; loading + inline error states
  - [x] **Delete account** — opens confirm sheet/dialog: 30-day policy copy, Cancel + destructive Confirm; on success redirect home
  - [x] Keep **Sign out** as last control (existing handler)

- [x] **i18n** — strings in `web/messages/en.json` under `account.export*` and `account.delete*`

- [x] **Out of scope (document only)**
  - [x] Automated hard-delete/anonymize job after 30 days — ops runbook note in story completion or `docs/ops/supabase-provisioning.md` § Account deletion (purge reviews → "Deleted user" per addendum §H)
  - [x] Privacy Policy / Terms rewrite → Story **15.3**

## Dev Notes

### PO / architecture alignment

| Topic | Decision |
|-------|----------|
| Export format | Single JSON object; ISO `exported_at`; nested trips |
| Delete Phase 2 scope | **Request + lock + sign-out** only; purge is manual/cron later |
| UGC on purge | Anonymize author display to "Deleted user"; keep review/comment text (addendum §H assumption) |
| Photos on purge | Remove from Storage buckets (`avatars`, `review-photos`, `passport-photos`) — purge job, not 9.5 |
| Legal pages | Story **15.3** updates Privacy Policy; 9.5 may link to `/privacy` |

### Export JSON shape (contract)

```json
{
  "exported_at": "2026-07-02T12:00:00.000Z",
  "profile": { "id", "username", "display_name", "avatar_url", "bio", "created_at", "updated_at" },
  "reviews": [],
  "comments": [],
  "trips": [{ "id", "title", "...", "days": [{ "day_number", "activities": [...] }] }],
  "passport_check_ins": [],
  "badges": [{ "badge_id", "earned_at" }],
  "saved_resorts": [{ "resort_slug", "saved_at" }]
}
```

Omit `role` from export profile (same as PATCH omission in 9.4). Omit internal storage paths from export unless needed for user data portability — include `photo_path` / review photo paths as stored in DB.

### Reuse from Story 9.4 (do not reinvent)

| Pattern | Location |
|---------|----------|
| Auth gate | `web/src/lib/auth/requireUser.ts` |
| API errors | `web/src/lib/api/errors.ts` — `createApiError`, `jsonOk` |
| Profile types | `web/src/lib/auth/profileTypes.ts` |
| Admin client | `web/src/lib/supabase/admin.ts` — server Route Handlers only |
| Account page shell | `AccountPageClient.tsx`, `AccountSettingsForm.tsx` |
| Analytics track | `web/src/lib/analytics/track.ts` + `phase2Events.ts` |

### Middleware / session

Middleware already refreshes session via `updateSession()`. Deletion lock is enforced at **callback + API** layers (do not block all traffic in middleware). After delete, client should hard-navigate to `/` so `AuthProvider` clears local profile state.

### Rate limiting

Apply existing `auth` policy (10/min/IP) to `/api/account/delete` if abuse risk; export GET may use same or no limit (single user data).

### UX references

- [Source: `_powri/planning-artifacts/ux-designs/ux-phase2/design.md` — AccountSettings destructive section]
- [Source: `_powri/planning-artifacts/ux-designs/ux-phase2/experience.md` — AccountSettingsForm, Account delete pending state]
- Destructive buttons: use existing semantic `.text-error` / `--color-error` from 9.4 for delete CTA; export is neutral/secondary outline

### Previous story intelligence (9.4)

- Default username pattern: `/^user-[a-f0-9]{12}$/i` — unrelated to delete but do not break username prompt flow
- `AccountPageClient` uses `key={profile.id}` — deletion redirect should unmount form cleanly
- Avatar auto-save pattern is separate; export/delete live in new danger zone below Save

### File structure (expected)

```
supabase/migrations/004_account_deletion.sql
scripts/verify-supabase-schema.mjs
web/src/app/api/account/export/route.ts
web/src/app/api/account/delete/route.ts
web/src/lib/auth/accountExport.ts + accountExport.test.ts
web/src/lib/auth/accountDeletion.ts + accountDeletion.test.ts   # optional split
web/src/lib/analytics/authAnalytics.ts                          # or dedicated accountAnalytics.ts
web/src/components/account/AccountDangerZone.tsx                # optional extract
web/src/components/account/DeleteAccountConfirmSheet.tsx
web/src/components/account/AccountSettingsForm.tsx              # wire danger zone
web/src/app/api/auth/callback/route.ts                          # deletion_pending guard
web/src/components/auth/AuthProvider.tsx                        # deletion_pending message
web/messages/en.json
web/src/lib/analytics/phase2Events.ts
docs/analytics/tracking-plan.md                                 # §3.1 instrumentation row
```

### References

- [Source: `_powri/planning-artifacts/epics/phase2/epics-phase2.md` — Story 9.5]
- [Source: `_powri/planning-artifacts/prds/phase2/shards/04-features-auth.md` — FR-8.4]
- [Source: `_powri/planning-artifacts/prds/phase2/addendum.md` — §H GDPR lifecycle]
- [Source: `_powri/planning-artifacts/architecture/phase2/architecture-phase2.md` — §5 API table]
- [Source: `_powri/implementation-artifacts/9-4-profile-bootstrap-account-settings.md` — account patterns]

## Dev Agent Record

### Agent Model Used

Composer (Cursor)

### Debug Log References

- PO decisions: analytics fires client-side before DELETE; repeat delete → 409 CONFLICT; export rate-limited with auth policy; review photos nested under each review.

### Completion Notes List

- Migration 004 adds `profiles.deletion_requested_at` and tightens owner update RLS while pending.
- `GET /api/account/export` assembles nested JSON (reviews with photos, trips with days/activities); empty categories as `[]`.
- `POST /api/account/delete` uses admin client to stamp deletion, ban 720h, global sign-out; client fires `account_deletion_requested` before API call.
- Auth callback blocks re-login when deletion pending; SignInSheet shows friendly copy via `?auth=deletion_pending`.
- Account danger zone: Export · Delete (confirm sheet) · Sign out; privacy note links to `/privacy`.
- Ops runbook note added for 30-day hard purge (manual/cron).
- PR gates: lint, build, test:unit (83), test:analytics, test:launch — all pass.

### File List

- supabase/migrations/004_account_deletion.sql
- scripts/verify-supabase-schema.mjs
- docs/analytics/tracking-plan.md
- docs/ops/supabase-provisioning.md
- web/messages/en.json
- web/src/middleware.ts
- web/src/app/api/account/export/route.ts
- web/src/app/api/account/delete/route.ts
- web/src/app/api/auth/callback/route.ts
- web/src/lib/auth/accountExport.ts
- web/src/lib/auth/accountExport.test.ts
- web/src/lib/auth/accountDeletion.ts
- web/src/lib/auth/accountDeletion.test.ts
- web/src/lib/auth/accountClient.ts
- web/src/lib/analytics/authAnalytics.ts
- web/src/lib/analytics/phase2Events.ts
- web/src/components/account/AccountDangerZone.tsx
- web/src/components/account/DeleteAccountConfirmSheet.tsx
- web/src/components/account/AccountSettingsForm.tsx
- web/src/components/auth/AuthProvider.tsx
- web/src/components/auth/SignInSheet.tsx

### Change Log

- 2026-07-02: Story 9.5 implementation — GDPR export, account deletion request flow, re-login block, account UI danger zone.

## Senior Developer Review (AI)

**Review date:** 2026-07-02
**Outcome:** Conditional Approve — 1 HIGH item to fix before merge; 5 MEDIUM + 7 LOW recorded in backlog.

### Summary

Architecture is correct: RLS guards, `requireUser()` on every route, admin client isolated to server Route Handlers, cookie-only JWT, `returnTo` sanitization, and idempotency guard on deletion. The main risk is a non-atomic deletion sequence that can leave an account in an unrecoverable partial-deletion state (TD-022). All other findings are hardening opportunities.

### Findings

- [!] **[HIGH — Block merge]** `POST /api/account/delete` non-atomic: stamp succeeds but ban failure has no rollback, leaving account deletion-stamped but not locked, active session still valid → TD-022
- [x] [Pass] `requireUser()` used on all three new routes (export, delete, profile)
- [x] [Pass] `account_deletion_requested` analytics fires before API call per PO decision (deliberate)
- [x] [Pass] Callback route blocks re-login when `deletion_requested_at` is set
- [x] [Pass] `idempotency guard (409 CONFLICT) on repeat deletion requests
- [~] [Note] `select('*')` on reviews/comments/passport/saved_resorts — TD-026
- [~] [Note] Avatar upload client-only MIME validation — TD-027
- [~] [Note] No rate limit on `PATCH /api/account/profile` — TD-024
- [~] [Note] HTTP 400 on server-side failures — TD-029
- [~] [Note] `bio` not profanity-filtered — TD-030
- [~] [Note] Large export no size cap — TD-033
