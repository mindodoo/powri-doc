---
baseline_commit: a6a25c828efc1e2906261f66f04dfa87656390a7
---

# Bug 9.11: Post-sign-in — no visual confirmation; new users not redirected to /account

Status: done  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`)  
**Severity:** P2 — UX issue; new users are not guided to complete their profile; sign-in silently lands on home page with no confirmation

---

## Symptom

After a successful Google OAuth sign-in on production:

1. User is redirected to the **home page** (or Vercel deployment root `/`)
2. There is **no visible confirmation** that sign-in succeeded
3. The user must independently notice a change in the navigation bar to confirm they are signed in
4. New users are never guided to `/account` to enter their display name or upload a profile picture

This was particularly disorienting in incognito mode (new user): the home page looks identical before and after sign-in. The user believed the sign-in had failed even though it had succeeded.

---

## Root cause

Two separate gaps in the post-sign-in flow:

**Gap 1 — No user-facing confirmation:**
The callback route sets `?auth=signed_in` on the redirect URL when sign-in completes. `AuthProvider` / `AuthSessionSync` consume this param to fire the `auth_sign_in_completed` analytics event, but they display no visible toast, banner, or any other user-facing indicator.

**Gap 2 — New user not redirected to `/account`:**
The callback route redirects to `returnTo` (default `/`) unconditionally. There is no logic to detect a first-time sign-in (`isNewUser`) and route new users to `/account` for profile completion. New users land on the home page with a default `user-{id}` username and no profile photo, without knowing they can or should customize their profile.

---

## Fix steps

### 1. Show a sign-in success toast
In `AuthProvider` or `AuthSessionSync`, when the `?auth=signed_in` query param is detected on mount:
- Show a brief dismissible toast notification: "Signed in successfully"
- Auto-dismiss after approximately 3 seconds
- This applies to all sign-in methods (Google OAuth + magic link)

### 2. Redirect new users to `/account`
In `web/src/app/api/auth/callback/route.ts`, after a successful `exchangeCodeForSession`:
- Call the `isNewUser` utility (already exists in `web/src/lib/auth/isNewUser.ts`) to detect first-time sign-in
- If `isNewUser === true` **and** no explicit `returnTo` path was requested (i.e. `returnTo` is `/` or absent), redirect to `/account?welcome=1` instead
- If `isNewUser === true` **and** there is an explicit `returnTo`, honour the `returnTo` (user was mid-flow when prompted to sign in)

### 3. Add a welcome prompt on `/account`
In `web/src/app/[locale]/account/page.tsx` (client component), detect `?welcome=1` on mount:
- Show a one-time welcome message: "Welcome to Powri! Add your name and a profile photo to get started."
- This prompt should disappear once the user saves their profile or navigates away

### 4. Returning users — no change
Returning users continue to be redirected to `returnTo` (default `/`) with no interruption. The sign-in toast applies to both new and returning users.

---

## Acceptance criteria

1. [x] After signing in via Google OAuth on `powri.vercel.app`, a toast "Signed in successfully" appears and auto-dismisses
2. [x] A new user completing their first Google OAuth sign-in is redirected to `/account?welcome=1`
3. [x] `/account` shows a welcome prompt when `?welcome=1` is present in the URL
4. [x] A returning user signs in and is redirected to their previous `returnTo` (or `/`), not to `/account`
5. [x] A new user who was mid-flow when prompted to sign in (explicit `returnTo`) is redirected to `returnTo`, not to `/account`
6. [x] The `auth_sign_in_completed` analytics event continues to fire on sign-in (no regression)

---

## Tasks / Subtasks

- [x] Add `buildPostSignInRedirectUrl` helper + unit tests (`postSignInRedirect.test.ts`)
- [x] Update auth callback route to redirect new users to `/account?welcome=1`
- [x] Add `AuthSuccessToast` and wire into `AuthProvider` on `?auth=signed_in`
- [x] Add welcome prompt in `AccountPageClient` for `?welcome=1`
- [x] Add i18n copy in `messages/en.json`
- [x] Add callback route integration tests (`route.test.ts`) + launch-gate check
- [x] Run `lint`, `build`, `test:unit`, `test:analytics` from `web/`

---

## Files involved

| File | Role |
|------|------|
| `web/src/app/api/auth/callback/route.ts` | Redirect target after sign-in; add `isNewUser` check |
| `web/src/components/auth/AuthProvider.tsx` | `?auth=signed_in` param handler; add toast |
| `web/src/components/auth/AuthSessionSync.tsx` | Session sync + analytics on sign-in |
| `web/src/lib/auth/isNewUser.ts` | Existing new-user detection utility |
| `web/src/app/[locale]/account/page.tsx` | Add `?welcome=1` handler and welcome prompt |
| `web/messages/en.json` | Toast copy + welcome prompt copy |

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Added `buildPostSignInRedirectUrl` to centralize post-sign-in redirect logic; new users with default `returnTo` (`/` or absent) land on `/account?welcome=1&auth=signed_in`.
- `AuthSuccessToast` shows "Signed in successfully" on `?auth=signed_in`, auto-dismisses after 3s with slide-up exit animation, entry slide-down + fade-in; dismissible manually.
- `AccountPageClient` shows a one-time welcome banner from `?welcome=1`; dismissed on profile save; URL param stripped on mount.
- `AuthSessionSync` unchanged — `auth_sign_in_completed` analytics continues to fire via existing `?auth=signed_in` handler.
- Regression guards: unit tests (`postSignInRedirect.test.ts`), integration tests (`route.test.ts`), launch-gate check (`verifyPostSignInRedirect`).
- **Manual verify (AC 1):** After deploy, confirm toast on production Google OAuth sign-in.

### File List

- `web/src/lib/auth/postSignInRedirect.ts` (added)
- `web/src/lib/auth/postSignInRedirect.test.ts` (added)
- `web/src/app/api/auth/callback/route.ts` (modified)
- `web/src/app/api/auth/callback/route.test.ts` (added)
- `web/src/components/auth/AuthSuccessToast.tsx` (added)
- `web/src/components/auth/AuthSuccessToast.module.css` (added)
- `web/src/components/auth/AuthProvider.tsx` (modified)
- `web/src/components/account/AccountPageClient.tsx` (modified)
- `web/messages/en.json` (modified)
- `web/scripts/verify-launch-gates.ts` (modified)

### Change Log

- 2026-07-08: Post-sign-in toast, new-user redirect to `/account?welcome=1`, welcome prompt (Story 9.11).
- 2026-07-08: Toast entry/exit animations via CSS module; deferred mount for reliable entry animation.
