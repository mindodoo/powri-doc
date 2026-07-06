---
baseline_commit: fd70081cef3daff59abc2dcb3a6f81acecdc375a
---

# Bug 9.12: OAuth `bad_oauth_state` error redirects to literal `/**` path → 404

Status: done  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`)  
**Severity:** P2 — OAuth failures produce a 404 with raw error codes in the URL; user has no recovery path

---

## Symptom

On production, when Google OAuth fails — most commonly observed in incognito mode on a first OAuth attempt — the browser is redirected to:

```
https://powri.vercel.app/**?error=invalid_request&error_code=bad_oauth_state&error_description=OAuth+state+has+expired
```

This URL produces a **404** because `/**` is a literal path segment — no route exists at that path in the Next.js app. The URL also exposes raw technical error codes (`bad_oauth_state`, `invalid_request`) in the address bar with no user-readable explanation and no recovery action.

---

## Root cause

Story 9.7 added `https://powri.vercel.app/**` to the **Supabase Redirect URLs** allow-list as a wildcard pattern intended to permit any production callback path. When an OAuth error occurs before the browser reaches the Powri callback route, Supabase redirects the browser to its stored redirect URL entry — using `https://powri.vercel.app/**` literally as the destination path, not as a glob pattern. The result is a redirect to the literal path `/**`, which is not a valid Next.js route and returns a 404.

### Production misconfiguration (confirmed 2026-07-06)

On `powri-prod`, the wildcard was set on the wrong field:

| Field | Incorrect (prod) | Correct |
|-------|------------------|---------|
| **Site URL** | `https://powri.vercel.app/**` | `https://powri.vercel.app` |
| **Redirect URLs** | `https://powri.vercel.app/api/auth/callback` | *(unchanged — already correct)* |

**Site URL** must be the origin only — no path, no `/**`. Supabase appends OAuth error query params to Site URL on failure; a literal `/**` in Site URL produces the `powri.vercel.app/**?error=…` 404 seen in production.

Remove `/**` from **Site URL** and from **Redirect URLs** if it appears in either list. Story 9.7’s Redirect-URL wildcard guidance was the original hypothesis; the live dashboard had the wildcard on Site URL instead.

---

## Fix steps

### 1. Fix `powri-prod` Supabase URL Configuration (ops)

In **Supabase Dashboard → `powri-prod` → Authentication → URL Configuration**:

| Field | Action |
|-------|--------|
| **Site URL** | Set to `https://powri.vercel.app` — **not** `https://powri.vercel.app/**` |
| **Redirect URLs** | Keep `https://powri.vercel.app/api/auth/callback`; remove `https://powri.vercel.app/**` if listed |

The specific callback path is all that is required for the PKCE flow. Site URL must be the origin only; wildcards are not supported and are treated as literal paths on OAuth errors.

Update [`docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md) to reflect the corrected configuration.

### 2. Add an `/auth/error` page

Create `web/src/app/[locale]/auth/error/page.tsx` as a user-friendly OAuth error landing page:
- Read `error`, `error_code`, and `error_description` from the URL query params
- Map known error codes to user-readable messages:

  | `error_code` | User-facing message |
  |--------------|---------------------|
  | `bad_oauth_state` | "Your sign-in session expired. Please try again." |
  | `access_denied` | "Sign-in was cancelled. You can try again whenever you're ready." |
  | Anything else | "Something went wrong during sign-in. Please try again." |

- Show a "Try signing in again" CTA that opens the `SignInSheet`
- Do not render raw `error_code` or `error_description` values in the page

### 3. Route Supabase OAuth errors to the error page

After removing the wildcard (step 1), Supabase OAuth error redirects will fall back to the **Site URL** (`https://powri.vercel.app`) with `?error=...&error_code=...&error_description=...` appended. Handle these params:

- In `AuthProvider` (which runs on every page), detect the presence of `?error_code=` on mount
- If detected, redirect to `/auth/error?error=...&error_code=...&error_description=...` so the user lands on the dedicated error page with clean URL handling
- Clear the params from the URL after redirect to avoid the error state persisting on back-navigation

### 4. Re-test

- Start Google OAuth in incognito, clear cookies mid-flow (or wait for state to expire), then complete → user lands on `/auth/error` with a user-friendly message and a "Try again" CTA; no 404
- Confirm that the `/**` literal redirect URL no longer appears in Supabase Dashboard
- Confirm that normal Google OAuth sign-in still works end-to-end (no regression from removing the wildcard)

---

## Acceptance criteria

1. [ ] `powri-prod` Site URL is `https://powri.vercel.app` (no `/**`) and `https://powri.vercel.app/**` is absent from Redirect URLs *(manual ops — PO)*
2. [x] An OAuth failure (e.g. `bad_oauth_state`) no longer produces a 404
3. [x] The user lands on a `/auth/error` page with a user-friendly message (not raw error codes in the URL or the page)
4. [x] A "Try signing in again" CTA is available on the error page
5. [ ] Normal Google OAuth sign-in continues to work end-to-end after the wildcard is removed *(manual — verify after ops + deploy)*
6. [x] `docs/qa/phase2/deploy-environment.md` is updated to reflect the corrected Redirect URLs list

## Tasks / Subtasks

- [x] Document corrected Redirect URLs in `deploy-environment.md` (no wildcard)
- [x] Add `oauthErrors` helper + unit tests (`oauthErrors.test.ts`)
- [x] Add `/auth/error` page + `AuthErrorPageClient` with i18n copy
- [x] Update `AuthProvider` to redirect `?error_code=` to `/auth/error`
- [x] Add E2E integration tests (`auth-oauth-error.spec.ts`) + launch-gate check
- [x] Run `lint`, `build`, `test:launch`, `test:unit`, `test:e2e` from `web/`

---

## Files involved

| File | Role |
|------|------|
| Supabase Dashboard (`powri-prod`) | Fix Site URL + remove `/**` from Redirect URLs if present |
| `web/src/app/[locale]/auth/error/page.tsx` | New — OAuth error landing page |
| `web/src/components/auth/AuthProvider.tsx` | Detect `?error_code=` on mount; redirect to error page |
| `web/messages/en.json` | OAuth error copy for the error page |
| `docs/qa/phase2/deploy-environment.md` | Update Redirect URLs documentation |

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Added `/auth/error` landing page with mapped user-facing copy; raw Supabase error params are consumed then stripped from the URL.
- `AuthProvider` redirects any page load with `?error_code=` to `/auth/error` via `getOAuthErrorRedirectPath`.
- Regression guards: unit tests (`oauthErrors.test.ts`), launch-gate check, E2E flow (`auth-oauth-error.spec.ts`).
- **Manual ops (AC 1, 5):** Set `powri-prod` **Site URL** to `https://powri.vercel.app` (remove `/**`); keep **Redirect URL** `https://powri.vercel.app/api/auth/callback`. Re-test Google OAuth E2E on production after deploy.

### File List

- `web/src/lib/auth/oauthErrors.ts` (added)
- `web/src/lib/auth/oauthErrors.test.ts` (added)
- `web/src/components/auth/AuthErrorPageClient.tsx` (added)
- `web/src/app/[locale]/auth/error/page.tsx` (added)
- `web/src/components/auth/AuthProvider.tsx` (modified)
- `web/messages/en.json` (modified)
- `web/e2e/auth-oauth-error.spec.ts` (added)
- `web/scripts/verify-launch-gates.ts` (modified)
- `docs/qa/phase2/deploy-environment.md` (modified)

### Change Log

- 2026-07-06: OAuth error landing page + AuthProvider redirect; document no-wildcard URL config (Story 9.12).
- 2026-07-06: Call out prod misconfiguration — `/**` on Site URL, not Redirect URLs.
