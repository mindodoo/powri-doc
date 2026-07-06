# Bug 9.6: Production sign-in redirects to localhost with auth_error

Status: done  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Fix story:** [`9-7-supabase-prod-staging-separation.md`](9-7-supabase-prod-staging-separation.md) — prod-only Supabase project + Vercel env split  
**Reported:** 2026-07-03 · **Environment:** Production  
**Severity:** P0 — blocks all production sign-in (Google OAuth + magic link)

---

## Symptom

On production, signing in via **Google OAuth** or **email magic link** always fails and redirects to:

`http://localhost:3000/?auth_error=1`

instead of the production domain.

---

## Root cause (confirmed by code trace)

The `auth_error=1` query param is **only** set in `web/src/app/api/auth/callback/route.ts` when:

1. The callback is hit **without** a `code` query param, or
2. `supabase.auth.exchangeCodeForSession(code)` returns an error.

The fail redirect uses the **callback request's origin**:

```ts
const origin = requestUrl.origin;
const failUrl = new URL(returnTo, origin);
failUrl.searchParams.set('auth_error', '1');
```

Landing on `http://localhost:3000/?auth_error=1` means Supabase completed auth but sent the browser to **`http://localhost:3000/api/auth/callback`** (not production). The callback ran on localhost, session exchange failed (PKCE verifier cookie lives on production domain), and the handler redirected to `/?auth_error=1` on localhost.

### Why both providers fail the same way

| Step | Google OAuth | Magic link |
|------|--------------|------------|
| App sends redirect | `buildAuthCallbackUrl(window.location.origin, returnTo)` in `SignInSheet.tsx` | `buildAuthCallbackUrl(origin, returnTo)` in `/api/auth/magic-link` using `request.url` origin |
| Supabase validates redirect | Must be in **Redirect URLs** allow list | Same |
| If not allowed / Site URL mismatch | Falls back to **Site URL** (`http://localhost:3000`) | Same |

App code correctly uses the current origin on production. The failure is **Supabase Dashboard URL configuration**, not client `redirectTo` construction.

### Evidence this was a known ops gap

- [`docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md) checklist item **unchecked**: “Supabase Site URL matches production domain (not localhost)”
- Story 9.3 manual E2E sign-in was never signed off (blocked on Dashboard OAuth setup)

---

## Fix steps

> **Preferred fix:** Complete Story **9.7** (separate `powri-prod` from staging). Steps below are the minimal prod auth config; 9.7 adds data isolation and prevents recurrence.

### 1. Supabase Dashboard (required — unblocks production)

**Authentication → URL Configuration:**

| Setting | Value |
|---------|--------|
| **Site URL** | `https://<production-domain>` (e.g. custom domain or primary Vercel URL) |
| **Redirect URLs** | Add all of: |
| | `https://<production-domain>/api/auth/callback` |
| | `https://<preview-domain>/api/auth/callback` (each Vercel preview host, or wildcard if supported) |
| | `http://localhost:3000/api/auth/callback` (keep for local dev) |

Use exact paths — Supabase matches the full callback URL including query string patterns for `returnTo`.

### 2. Google Cloud Console (verify, usually already correct)

**OAuth client → Authorized redirect URIs** must include Supabase's broker URL:

`https://<project-ref>.supabase.co/auth/v1/callback`

(Powri's `/api/auth/callback` is **not** registered in Google Console — Supabase handles that hop.)

### 3. Re-test on production

- [x] Google sign-in → returns to production `/?auth=signed_in` (or intended `returnTo`) *(PO verified 2026-07-03)*
- [x] Magic link from production → link in email points to production `/api/auth/callback`, not localhost *(PO verified 2026-07-03)*
- [ ] Local dev still works with localhost redirect URL in allow list

### 4. Optional code hardening (follow-up, not required for ops fix)

- Log `exchangeCodeForSession` error server-side in callback route (no PII) for future debugging
- Handle `?auth_error=1` in `AuthProvider` with user-facing copy (currently no UI handler)
- Consider `NEXT_PUBLIC_APP_URL` for server-side `emailRedirectTo` if preview/proxy host headers ever diverge from public URL

---

## Acceptance criteria

1. [x] Production Google OAuth completes; user lands on production domain with session established *(2026-07-03)*
2. [x] Production magic link email contains production callback URL; completion succeeds *(2026-07-03)*
3. [x] Supabase prod **Site URL** + **Redirect URLs** verified on `powri-prod` (`https://powri.vercel.app`)
4. [ ] Story 9.3 manual E2E row checked off in `9-3-sign-in-sheet-google-magic-link.md` *(deferred)*

---

## Files involved (reference)

| File | Role |
|------|------|
| `web/src/app/api/auth/callback/route.ts` | Sets `auth_error=1`; uses `requestUrl.origin` for redirect |
| `web/src/components/auth/SignInSheet.tsx` | OAuth `redirectTo` via `window.location.origin` |
| `web/src/app/api/auth/magic-link/route.ts` | Magic link `emailRedirectTo` via request origin |
| `web/src/lib/auth/returnTo.ts` | Builds `/api/auth/callback?returnTo=…` |
| `docs/qa/phase2/deploy-environment.md` | Ops checklist (Site URL / Redirect URLs) |

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- **Resolved 2026-07-03** via Story **9.7**: separate `powri-prod` (`ysbqqzreshsjpgenqijh`) + Vercel Production env split.
- Prod auth fix required **`powri-prod` Redirect URLs** to include query-string callback:
  - `https://powri.vercel.app/api/auth/callback?returnTo=/`
  - `https://powri.vercel.app/**`
- PO verified Google OAuth and magic link no longer redirect to `localhost:3000`.
- **Follow-up:** PO reported odd post-login UX flow — separate bug story to be filed (out of scope for 9.6).
