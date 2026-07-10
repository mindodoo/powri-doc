# Story 9.1: Supabase Project & Database Migration

Status: done

<!-- Validation optional: bmad-create-story before bmad-dev-story -->

## Story

As a **developer**,
I want Supabase provisioned with Phase 2 schema and RLS,
So that all UGC features have a secure data foundation.

## Acceptance Criteria

1. **Given** a Supabase project is created  
   **When** migrations [`001_phase2_core.sql`](../../supabase/migrations/001_phase2_core.sql) and [`002_storage_buckets.sql`](../../supabase/migrations/002_storage_buckets.sql) are applied  
   **Then** all tables exist: `profiles`, `reviews`, `review_photos`, `comments`, `passport_check_ins`, `user_badges`, `trips`, `trip_days`, `trip_activities`, `saved_resorts`, `content_reports`, `moderation_audit`, `places_cache`

2. **And** RLS is enabled on user-facing tables per migration 001

3. **And** Storage buckets `review-photos` and `passport-photos` exist (migration 002)

4. **And** `web/.env.example` documents Supabase vars (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`) and auth redirect URL notes — **committed to git** (not gitignored)

5. **And** Vercel Preview + Production env var checklist updated (same three Supabase keys; Root Directory `web`)

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [x] **Schema verify script:** `npm run verify:supabase` (repo root) passes against linked project using `web/.env.local` keys
- [x] **Git:** `web/.env.example` tracked (not ignored by `.env*`)
- [x] **Manual:** Supabase Dashboard → Storage shows both buckets; SQL Editor table list matches AC #1 (covered by verify script REST checks)
- [x] **Manual:** Vercel env vars set for Preview + Production (PO confirmed 2026-07-02)
- [x] **Unit / analytics / E2E:** N/A — infra-only story
- [x] **PR gates (epic complete):** deferred to epic PR

## Tasks / Subtasks

- [x] **Branch** (before code)
  - [x] Create `feature/epic-9-supabase-auth` from updated `main`

- [x] **Fix env example visibility** (AC: 4)
  - [x] Update `web/.gitignore`: ignore `.env*` but **negate** `!.env.example`
  - [x] Ensure `web/.env.example` matches architecture §10
  - [x] Stage `web/.env.example` for commit

- [x] **Provision / apply migrations** (AC: 1–3)
  - [x] Reuse linked project ref `dbeujcltupsccxxhfuji` (PO decision)
  - [x] Cloud schema verified via `npm run verify:supabase` (migrations already applied)
  - [x] Optional: `npx supabase db push` after local `supabase login` if schema drifts

- [x] **Automated verification** (AC: 1–3, DoD)
  - [x] Add `scripts/verify-supabase-schema.mjs`
  - [x] Add root `package.json` script: `"verify:supabase"`
  - [x] Document in `docs/ops/supabase-provisioning.md` § Verify

- [x] **Deploy documentation** (AC: 5)
  - [x] Add [`docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md) with Supabase Vercel checklist
  - [x] Link from `docs/qa/phase2/README.md` and `docs/README.md`

- [x] **Out of scope — defer to later stories**
  - [x] Auth dashboard config → Story **9.3** (PO decision)

## Dev Notes

### PO decisions (2026-07-02)

| Decision | Choice |
|----------|--------|
| Supabase project | Reuse `dbeujcltupsccxxhfuji` |
| Vercel env vars | PO set Preview + Production manually |
| Auth providers (Google + Email) | Story **9.3** |
| Branch | `feature/epic-9-supabase-auth` |

### References

- [Source: `_powri/planning-artifacts/epics/phase2/epics-phase2.md` — Story 9.1]
- [Source: `_powri/planning-artifacts/architecture/phase2/architecture-phase2.md` §3, §10]
- [Source: `docs/ops/supabase-provisioning.md`]
- [Source: `docs/qa/phase2/deploy-environment.md`]

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npx supabase db push` failed in agent sandbox (no CLI access token); REST verify against existing cloud schema succeeded.

### Completion Notes List

- Fixed `web/.gitignore` so `.env.example` is committed.
- Added `npm run verify:supabase` — 13 tables + 2 buckets OK on `dbeujcltupsccxxhfuji`.
- RLS defined in migration 001; not separately probed (service role bypasses RLS by design).

### File List

- `web/.gitignore` (modified)
- `web/.env.example` (added to git)
- `scripts/verify-supabase-schema.mjs` (new)
- `package.json` (modified — `verify:supabase`)
- `docs/ops/supabase-provisioning.md` (modified)
- `docs/qa/phase2/deploy-environment.md` (new)
- `docs/qa/phase2/README.md` (modified)
- `docs/README.md` (modified)
- `_powri/implementation-artifacts/Epic-9/9-1-supabase-project-database-migration.md` (new)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-07-02  
**Outcome:** Approve

### Summary

Infra-only story with minimal, focused diff. Verify script uses service role against PostgREST — appropriate for schema existence checks. No app code or secrets committed.

### Findings

- [x] [Pass] All 5 ACs satisfied
- [x] [Pass] `web/.env.example` no longer gitignored
- [x] [Pass] Deploy checklist documents Vercel vars; PO sign-off recorded
- [x] [Pass] Auth dashboard deferred to 9.3 per PO
- [x] [Note] RLS not runtime-tested here — covered by migration SQL + future Epic 12 smoke (epic validate gate)
