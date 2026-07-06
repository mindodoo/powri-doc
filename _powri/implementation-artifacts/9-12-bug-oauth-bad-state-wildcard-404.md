# Bug 9.12: OAuth `bad_oauth_state` error redirects to literal `/**` path → 404

Status: ready-for-dev  
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

---

## Fix steps

### 1. Remove the wildcard from `powri-prod` Supabase Redirect URLs (ops)

In **Supabase Dashboard → `powri-prod` → Authentication → URL Configuration → Redirect URLs**:

- **Remove:** `https://powri.vercel.app/**`
- **Keep (or add if missing):** `https://powri.vercel.app/api/auth/callback`

The specific callback path is all that is required for the PKCE flow. Removing the wildcard prevents Supabase from using it as a literal error redirect target.

Update [`docs/qa/phase2/deploy-environment.md`](../../docs/qa/phase2/deploy-environment.md) to reflect the corrected Redirect URLs list.

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

1. [ ] `https://powri.vercel.app/**` is removed from `powri-prod` Supabase Redirect URLs
2. [ ] An OAuth failure (e.g. `bad_oauth_state`) no longer produces a 404
3. [ ] The user lands on a `/auth/error` page with a user-friendly message (not raw error codes in the URL or the page)
4. [ ] A "Try signing in again" CTA is available on the error page
5. [ ] Normal Google OAuth sign-in continues to work end-to-end after the wildcard is removed
6. [ ] `docs/qa/phase2/deploy-environment.md` is updated to reflect the corrected Redirect URLs list

---

## Files involved

| File | Role |
|------|------|
| Supabase Dashboard (`powri-prod`) | Remove `/**` wildcard from Redirect URLs |
| `web/src/app/[locale]/auth/error/page.tsx` | New — OAuth error landing page |
| `web/src/components/auth/AuthProvider.tsx` | Detect `?error_code=` on mount; redirect to error page |
| `web/messages/en.json` | OAuth error copy for the error page |
| `docs/qa/phase2/deploy-environment.md` | Update Redirect URLs documentation |

---

## Dev Agent Record

### Agent Model Used

_to be filled_

### Completion Notes

_to be filled_
