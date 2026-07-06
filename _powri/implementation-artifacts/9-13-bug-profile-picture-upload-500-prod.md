# Bug 9.13: Profile picture upload returns 500 Internal Server Error on production

Status: ready-for-dev  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`) only — works on `localhost:3000`  
**Severity:** P1 — users cannot set a profile picture on production; `/account` upload flow is broken

---

## Symptom

On `powri.vercel.app`, attempting to upload a profile picture on the `/account` page returns a **500 Internal Server Error**. The UI shows an error but gives no actionable explanation. The identical flow works correctly on `localhost:3000`.

---

## Root cause (hypothesis)

Story 9.7 created a separate `powri-prod` Supabase project. The `avatars` storage bucket and its RLS policies are created by migration `003_avatars_bucket.sql` (introduced in Story 9.4). If this migration was not applied to `powri-prod`, or if the bucket RLS policies differ from staging, uploads will fail with a server-side error.

Story 9.7 notes that `npm run verify:supabase` was run against prod keys and passed — however, the schema verify script may not cover all bucket permission checks needed for upload (INSERT/UPDATE), only bucket existence.

**Likely candidates in priority order:**

1. `003_avatars_bucket.sql` was applied to `powri-prod` but the **RLS policies** for authenticated upload were not created or differ from staging
2. The `avatars` bucket on `powri-prod` exists but is **not public** (required for serving uploaded avatars via URL)
3. `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY` on Vercel Production env vars are correct but the **storage policy** does not allow the `anon` or `authenticated` role to INSERT into the bucket
4. The migration `003_avatars_bucket.sql` was not applied to `powri-prod` at all (least likely given verify passed, but worth confirming)

---

## Fix steps

### 1. Add server-side logging to capture the exact error

In `web/src/lib/auth/uploadAvatar.ts` (or wherever the Supabase Storage upload call is made), log the Supabase storage error detail (error code + message, no PII) before returning the 500. Deploy to production and re-trigger the upload to see the exact error in Vercel logs.

This one step will immediately narrow down whether the cause is bucket missing, RLS rejection, or something else.

### 2. Verify `avatars` bucket and RLS policies on `powri-prod`

In **Supabase Dashboard → `powri-prod` → Storage → Buckets**:
- Confirm the `avatars` bucket exists
- Confirm the bucket is set to **Public** (avatars are served by URL to display in the UI)

In **Supabase Dashboard → `powri-prod` → Storage → Policies**:
- Confirm policies exist that allow:
  - `authenticated` role: **INSERT** objects under `{user.id}/`
  - `authenticated` role: **UPDATE** their own objects
  - `public` / `anon` role: **SELECT** (read) objects (for displaying avatars)
- Cross-reference with `supabase/migrations/003_avatars_bucket.sql` to ensure all policies were reproduced

### 3. Apply missing migration or policies

If the bucket or policies are missing, apply via SQL Editor in Supabase Dashboard (`powri-prod`) or via:

```bash
# One-off with prod DB URL
npx supabase db push --db-url <powri-prod-db-url>
```

Alternatively, copy the specific `CREATE POLICY` statements from `003_avatars_bucket.sql` and run them directly in the SQL Editor.

### 4. Update `verify:supabase` to cover upload permissions

After the fix, update `scripts/verify-supabase-schema.mjs` to also check that the `avatars` bucket policies allow authenticated upload — so this class of misconfiguration is caught in future prod provisioning.

### 5. Re-test on production

- Sign in on `powri.vercel.app` → `/account` → upload a JPG under 2 MB → photo appears immediately and persists on reload
- Upload a PNG and WebP — both succeed
- Upload a file over 2 MB → inline error shown under upload button (not a 500)
- Upload a non-image file type → inline error shown

---

## Acceptance criteria

1. [ ] `npm run verify:supabase` passes with `powri-prod` keys, including bucket existence check
2. [ ] Profile picture upload on `powri.vercel.app` succeeds for a signed-in user (JPG/PNG/WebP < 2 MB)
3. [ ] Uploaded photo auto-saves and is visible immediately without a page reload
4. [ ] Uploaded photo persists after page reload
5. [ ] Oversize file (> 2 MB) shows the inline validation error under the upload button — not a 500
6. [ ] Wrong file type shows the inline validation error under the upload button — not a 500
7. [ ] `verify:supabase` script extended to catch bucket policy misconfiguration in future prod provisioning

---

## Files involved

| File | Role |
|------|------|
| `supabase/migrations/003_avatars_bucket.sql` | Creates `avatars` bucket + RLS policies |
| `scripts/verify-supabase-schema.mjs` | `verify:supabase` script — extend to check upload policies |
| `web/src/lib/auth/uploadAvatar.ts` | Supabase Storage upload call — add error logging |
| `web/src/app/api/account/profile/route.ts` | PATCH handler that saves `avatar_url` after upload |
| Supabase Dashboard (`powri-prod`) | Storage → `avatars` bucket + RLS policies |

---

## Dev Agent Record

### Agent Model Used

_to be filled_

### Completion Notes

_to be filled_
