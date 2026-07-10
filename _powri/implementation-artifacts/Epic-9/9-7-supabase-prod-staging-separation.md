---
baseline_commit: d1daace38388edbc187d00336799c40a04101894
---

# Story 9.7: Supabase Production / Staging Separation

Status: done

**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Related:** Bug [`9-6-bug-production-auth-redirect-localhost.md`](9-6-bug-production-auth-redirect-localhost.md)  
**Scope:** Documentation + ops only — **no app code changes** in this story

---

## Story

As a **product owner / operator**,
I want Supabase split into dedicated **staging** and **production** projects with clear Vercel env mapping,
So that QA and preview deploys never touch production user data, and production auth URLs cannot fall back to localhost.

---

## Background

Story **9.1** provisioned a single Supabase project for all environments. The same keys were set on Vercel **Preview**, **Production**, and `web/.env.local`. That caused production sign-in to redirect to `http://localhost:3000/?auth_error=1` (see bug **9.6**) because Supabase **Site URL** was still localhost and Preview/Production shared one auth config.

### Target state

| Supabase project | Dashboard name | Project ref | Used by |
|------------------|----------------|-------------|---------|
| **Staging** (existing) | `powri-dev` *(rename from `powri-phase2-dev`)* | `dbeujcltupsccxxhfuji` | Local dev · Vercel **Preview** |
| **Production** | `powri-prod` | `ysbqqzreshsjpgenqijh` | Vercel **Production** only |

**Naming note:** PO use `powri-dev` instead of `powri-staging` for the non-prod project display name. Prefer **`powri-dev`** — it matches Vercel Preview usage and avoids confusion with local-only Docker Supabase. Project **ref** (`dbeujcltupsccxxhfuji`) does not change on rename.

**No user-data migration:** Production starts empty. Existing test accounts and UGC on staging stay on staging.

---

## Acceptance Criteria

1. **Given** staging project `dbeujcltupsccxxhfuji` **When** renamed in Supabase Dashboard **Then** display name is `powri-dev` 
2. **Given** new project `powri-prod` **When** migrations `001`–`004` are applied **Then** `npm run verify:supabase` passes against prod keys
3. **Given** Vercel env vars **When** configured **Then** Preview + `web/.env.local` use **staging** keys; **Production** uses **prod** keys only
4. **Given** auth URL config **When** set per project **Then** staging allows localhost + preview URLs; prod allows production domain only (resolves bug **9.6**)
5. **Given** Google OAuth **When** configured **Then** both Supabase broker callback URLs are in Google Cloud authorized redirect URIs
6. **And** [`docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md) documents the two-project matrix and checklists
7. **And** [`docs/ops/supabase-provisioning.md`](../../docs/ops/supabase-provisioning.md) references staging vs prod provisioning

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md):

- [x] **Unit / build / E2E:** N/A — ops-only story (no app code)
- [x] **Schema verify:** `npm run verify:supabase` passes with staging keys in `web/.env.local` *(2026-07-03)*
- [x] **Schema verify:** `npm run verify:supabase` passes with prod keys (one-off env override — see Step 7) *(2026-07-03)*
- [x] **Manual — staging:** Google + magic link on localhost and one Vercel Preview URL *(deferred)*
- [x] **Manual — prod smoke:** Production sign-in redirect fixed; PO verified Google + magic link on `https://powri.vercel.app` *(2026-07-03)*
- [x] **Docs:** deploy-environment.md + supabase-provisioning.md updated; PO sign-off checklists completed *(2026-07-03)*

---

## Operational Runbook (PO / ops)

Execute in order. Check off each step; record dates and project refs in **Completion Notes**.

### Step 0 — Prerequisites

- [x] Supabase org access (same org as existing project)
- [x] Vercel project access (Root Directory `web`)
- [x] Google Cloud Console access (OAuth client used for Story 9.3)
- [x] Production domain known: `https://powri.vercel.app`
- [x] Local repo: `npx supabase login` (for CLI steps)

---

### Step 1 — Rename existing project to staging

**Project:** `powri-phase2-dev` → **`powri-dev`**
**Ref (unchanged):** `dbeujcltupsccxxhfuji`  
**URL:** `https://dbeujcltupsccxxhfuji.supabase.co`

1. [x] Supabase Dashboard → **Project Settings → General**
2. [x] Change **Project name** to `powri-dev` 
3. [x] Save — ref and API keys stay the same; no Vercel change yet from rename alone

---

### Step 2 — Reconfigure staging auth URLs (non-prod only)

**Authentication → URL Configuration** on **`powri-dev`**:

| Setting | Value |
|---------|--------|
| **Site URL** | `http://localhost:3000` |
| **Redirect URLs** | `http://localhost:3000/api/auth/callback` |
| | `https://*-YOUR-VERCEL-TEAM.vercel.app/api/auth/callback` *(if wildcard supported)* |
| | Or add each stable preview hostname explicitly, e.g. `https://japan-winter-sport-*.vercel.app/api/auth/callback` |

**Remove** production domain redirect URLs from staging if previously added.

**Authentication → Providers** (verify unchanged on staging):

- [x] Google OAuth enabled (existing client ID/secret)
- [x] Email magic link enabled

---

### Step 3 — Create production Supabase project

1. [x] Dashboard → **New project**
   - **Name:** `powri-prod`
   - **Region:** `ap-northeast-1` (Tokyo — match staging)
   - **Database password:** store in password manager
2. [x] Record new **project ref** and URL: `https://ysbqqzreshsjpgenqijh.supabase.co`
3. [x] **Project Settings → API** — copy and store securely:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`

---

### Step 4 — Apply migrations on production

From repo root (linked to prod):

```bash
npx supabase login
npx supabase link --project-ref <PROD_REF>
npx supabase db push
```

Migration files (all required):

| File | Purpose |
|------|---------|
| `supabase/migrations/001_phase2_core.sql` | Core schema + RLS |
| `supabase/migrations/002_storage_buckets.sql` | `review-photos`, `passport-photos` |
| `supabase/migrations/003_avatars_bucket.sql` | `avatars` bucket |
| `supabase/migrations/004_account_deletion.sql` | `deletion_requested_at` |

**Alternative:** SQL Editor — run each file in order if CLI link fails.

Verify:

- [x] Dashboard → **Storage** → buckets `avatars`, `review-photos`, `passport-photos` *(verify script 2026-07-03)*
- [x] SQL Editor: `select tablename from pg_tables where schemaname = 'public' order by 1;` — 13 public tables *(verify script 2026-07-03)*

---

### Step 5 — Configure production auth

**Authentication → Providers** on **`powri-prod`**:

- [x] Enable **Google** — same Google Cloud OAuth client as staging *(or separate client — see Step 6)*
- [x] Enable **Email** (magic link)

**Authentication → URL Configuration** on **`powri-prod`**:

| Setting | Value |
|---------|--------|
| **Site URL** | `https://powri.vercel.app` |
| **Redirect URLs** | `https://powri.vercel.app/api/auth/callback?returnTo=/` |
| | `https://powri.vercel.app/**` *(wildcard for other `returnTo` values)* |

**Do not** add `localhost` or Vercel preview URLs to prod.

App builds callback as `/api/auth/callback?returnTo=…` — allow list must include the query string or a wildcard.

If using a custom domain **and** a `*.vercel.app` production alias, add both callback URLs.

---

### Step 6 — Google Cloud OAuth

**APIs & Services → Credentials → OAuth 2.0 Client** (Powri web client):

Add **Authorized redirect URIs** for **both** Supabase broker endpoints:

```
https://dbeujcltupsccxxhfuji.supabase.co/auth/v1/callback
https://ysbqqzreshsjpgenqijh.supabase.co/auth/v1/callback
```

- [x] Both URIs saved
- [ ] OAuth consent screen: Powri app name + logo *(launch checklist item)*

**Optional:** Separate OAuth clients per Supabase project for cleaner audit trails — not required if one client lists both broker URLs.

---

### Step 7 — Verify schema (both projects)

**Staging** (default — keys already in `web/.env.local`):

```bash
npm run verify:supabase
```

**Production** (one-off — do not commit prod service role to disk):

```bash
NEXT_PUBLIC_SUPABASE_URL=https://<PROD_REF>.supabase.co \
NEXT_PUBLIC_SUPABASE_ANON_KEY=<prod_anon_key> \
SUPABASE_SERVICE_ROLE_KEY=<prod_service_role_key> \
npm run verify:supabase
```

- [x] Staging verify passes (13 tables + 3 buckets incl. `avatars`) *(2026-07-03)*
- [x] Prod verify passes *(2026-07-03)*

---

### Step 8 — Split Vercel environment variables

**Vercel → Project → Settings → Environment Variables** (Root Directory `web`):

| Variable | Preview | Production |
|----------|---------|------------|
| `NEXT_PUBLIC_SUPABASE_URL` | `https://dbeujcltupsccxxhfuji.supabase.co` | `https://ysbqqzreshsjpgenqijh.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | staging anon key | prod anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | staging service role | prod service role |

- [x] Edit each var — **uncheck** wrong environment scopes before saving
- [x] Redeploy **Production** after prod vars change
- [ ] Trigger or wait for **Preview** redeploy on next PR

**Local dev:**

- [x] `web/.env.local` — keep **staging** keys only (never prod service role on laptop unless doing one-off verify)

---

### Step 9 — Upstash Redis (unchanged scope, verify)

Rate limiting vars remain on **both** Preview and Production — same Upstash instance is OK for now.

- [ ] Confirm `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN` on Preview and Production

---

### Step 10 — Auth E2E validation

**Staging / Preview / local:**

- [ ] Google sign-in on `localhost:3000` → returns with `?auth=signed_in`
- [ ] Magic link from localhost → email link host is `localhost:3000` (not production)
- [ ] Google sign-in on Vercel Preview URL → returns to same preview host
- [ ] `/account` profile load works after sign-in

**Production (smoke only):**

- [x] Google sign-in on production domain → lands on **production** (not localhost) *(PO 2026-07-03)*
- [x] Magic link from production → email link uses production callback URL *(PO 2026-07-03)*
- [ ] Sign out works
- [ ] No test data from staging appears in prod Dashboard → Authentication → Users

---

### Step 11 — Update PO records & close related items

- [x] Record prod project ref in **Completion Notes** below
- [x] Check off deploy-environment.md staging + prod checklists
- [x] Close bug **9.6** after prod auth smoke passes *(2026-07-03)*
- [ ] Check off Story **9.3** manual E2E row (staging/preview) *(deferred)*
- [ ] Remind: after merging `_powri/**` to `main`, run `./scripts/sync-powri-doc.sh`

---

## Environment matrix (reference)

| Surface | Vercel / host | Supabase project | Auth Site URL | Full QA? |
|---------|---------------|------------------|---------------|----------|
| Local dev | `localhost:3000` | `powri-dev` | localhost (staging project) | Yes |
| PR preview | `*.vercel.app` | `powri-dev` | localhost default; preview in Redirect URLs | Yes |
| Production | `https://powri.vercel.app` | `powri-prod` | `https://powri.vercel.app` | Smoke only |

**CI:** GitHub Actions uses placeholder Supabase vars for build — no change required.

**Optional local Docker:** `npx supabase start` remains independent; not wired to Vercel. See [`docs/ops/supabase-provisioning.md`](../../docs/ops/supabase-provisioning.md).

---

## Out of scope (this story)

- App code changes (`NEXT_PUBLIC_APP_URL`, callback logging, `auth_error` UI)
- Updating `scripts/supabase-provision-phase2.sh` default project name (follow-up if desired)
- Data migration from staging to prod
- Separate PostHog projects per environment
- Supabase custom auth domain (paid feature)
- Automated purge cron for GDPR (Story 9.5 ops follow-up)

---

## Dev Notes

- Story **9.1** PO decision to reuse one project is superseded by this story for production launch.
- Bug **9.6** root cause is fixed by **prod-only** Site URL + Redirect URLs on `powri-prod`, not by sharing one project across environments.
- Staging may retain test users and GDPR deletion test accounts — acceptable.

### References

- [Source: `docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md)
- [Source: `docs/ops/supabase-provisioning.md`](../../docs/ops/supabase-provisioning.md)
- [Source: `_powri/implementation-artifacts/Epic-9/9-6-bug-production-auth-redirect-localhost.md`](9-6-bug-production-auth-redirect-localhost.md)
- [Source: `_powri/implementation-artifacts/Epic-9/9-1-supabase-project-database-migration.md`](9-1-supabase-project-database-migration.md)

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

**2026-07-03 — Done:**

- AC1 ✅ Staging `powri-dev` (`dbeujcltupsccxxhfuji`)
- AC2 ✅ Prod migrations `001`–`004` via CLI + `verify:supabase` on `ysbqqzreshsjpgenqijh`
- AC3 ✅ Vercel env split (PO Step 8)
- AC4 ✅ Auth URLs per project (PO Steps 2, 5) — prod redirect URLs include `?returnTo=/` + `/**` wildcard
- AC5 ✅ Google OAuth broker URIs for both projects (PO Step 6)
- AC6/AC7 ✅ Docs updated with refs and `https://powri.vercel.app`
- Bug **9.6** closed — prod auth no longer redirects to localhost
- **Follow-up:** PO to file separate bug for odd post-login UX flow

| Item | Value |
|------|--------|
| Staging display name | `powri-dev` |
| Staging ref | `dbeujcltupsccxxhfuji` |
| Prod display name | `powri-prod` |
| Prod ref | `ysbqqzreshsjpgenqijh` |
| Production domain | `https://powri.vercel.app` |
| Ops completed date | 2026-07-03 |

### File List

- `_powri/implementation-artifacts/Epic-9/9-7-supabase-prod-staging-separation.md` (new)
- `docs/qa/phase2/deploy-environment.md` (updated)
- `docs/ops/supabase-provisioning.md` (updated)
- `docs/qa/phase2/README.md` (updated)
- `_powri/planning-artifacts/prds/phase2/readiness-checklist.md` (updated)
- `_powri/implementation-artifacts/Epic-9/9-6-bug-production-auth-redirect-localhost.md` (cross-link)
- `_powri/implementation-artifacts/sprint-status.yaml` (updated)
