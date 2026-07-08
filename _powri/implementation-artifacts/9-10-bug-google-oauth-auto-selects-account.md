---
baseline_commit: c0484178998bfa90791b949288b34cb630d2e470
---

# Bug 9.10: Google OAuth auto-selects account without showing picker

Status: review  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`)  
**Severity:** P2 — UX issue; users with multiple Google accounts cannot control which account is used to sign in to Powri

---

## Symptom

On production, in Chrome (regular or incognito) where a Google account has been previously used in the browser session:

1. User clicks "Sign in with Google"
2. Button changes to "Redirecting…"
3. Google completes the OAuth flow and signs the user in using the **browser's most recently used Google account** — the Google account picker is never shown
4. The user did not type anything or make any choice; Powri and Google decided on their behalf

This was observed in two separate scenarios:
- **Regular Chrome** (saved Google account from Gmail): auto-selected the account associated with Gmail open in another tab
- **Incognito Chrome** (Google account used earlier in the same incognito session): auto-selected that most recently used account

---

## Root cause

The `signInWithOAuth` call for Google in `SignInSheet.tsx` does not pass `queryParams: { prompt: 'select_account' }`. Without this parameter, Google's OAuth service checks whether the browser has an active or recent Google session and auto-selects it via the FedCM (Federated Credential Management) API or Google's standard account hint mechanism — bypassing the account selection screen.

---

## Fix steps

### 1. Add `prompt: 'select_account'` to Google OAuth call

In `web/src/components/auth/SignInSheet.tsx`, update the `signInWithOAuth` invocation to always force the Google account selection screen:

```ts
await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: buildAuthCallbackUrl(window.location.origin, returnTo),
    queryParams: {
      prompt: 'select_account',
    },
  },
})
```

`prompt: 'select_account'` instructs Google to always display the account chooser, even when there is exactly one signed-in account in the browser. This is the standard practice for apps where users may have multiple accounts.

### 2. Re-test on production

- **Regular Chrome, one saved Google account** → Google account picker appears with that account listed; user must click to confirm
- **Regular Chrome, multiple Google accounts** → picker lists all accounts; user selects one
- **Incognito, Google account used earlier in session** → picker appears; user selects
- **Incognito, no previous Google session** → picker appears with "Use another account" option
- **After selecting account** → sign-in completes normally and user lands on `/account` (new) or home (returning)

---

## Acceptance criteria

1. [ ] Clicking "Sign in with Google" on `powri.vercel.app` always opens the Google account selection screen *(manual — verify after deploy)*
2. [ ] The account picker is shown even when Chrome has exactly one saved Google account *(manual — verify after deploy)*
3. [ ] The account picker is shown in incognito mode when a Google account was used earlier in the same session *(manual — verify after deploy)*
4. [ ] The user explicitly selects their account before Powri proceeds with sign-in *(manual — verify after deploy)*
5. [ ] After account selection, sign-in completes normally — no regression on the OAuth flow *(manual — verify after deploy)*

## Tasks / Subtasks

- [x] Add `queryParams: { prompt: 'select_account' }` to Google `signInWithOAuth` in `SignInSheet.tsx`
- [x] Extract `buildGoogleOAuthSignInOptions()` helper + unit tests
- [x] Add E2E regression test (`e2e/auth-google-oauth.spec.ts`) + launch-gate check
- [x] Run `lint`, `build`, `test:launch`, `test:unit`, `test:e2e` from `web/`

---

## Files involved

| File | Role |
|------|------|
| `web/src/components/auth/SignInSheet.tsx` | Google `signInWithOAuth` call — add `queryParams` |

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Added `queryParams: { prompt: 'select_account' }` via `buildGoogleOAuthSignInOptions()` — forces Google's account chooser on every sign-in attempt, even when the browser has a single active Google session.
- Regression guards: unit tests (`googleOAuth.test.ts`), launch-gate check (`verify-launch-gates.ts`), E2E authorize-request assertion (`auth-google-oauth.spec.ts`).
- `lint`, `build`, `test:launch`, `test:unit`, and `test:e2e` pass on branch `fix/9-10-google-oauth-account-picker` (baseline `origin/main` @ `c048417`).
- Staging manual tests passed; AC 1–5 require manual production verification after merge/deploy.

### File List

- `web/src/lib/auth/googleOAuth.ts` (added)
- `web/src/lib/auth/googleOAuth.test.ts` (added)
- `web/e2e/auth-google-oauth.spec.ts` (added)
- `web/src/components/auth/SignInSheet.tsx` (modified)
- `web/scripts/verify-launch-gates.ts` (modified)
- `web/playwright.config.ts` (modified)

### Change Log

- 2026-07-06: Force Google account picker via `prompt: 'select_account'` on OAuth sign-in.
- 2026-07-06: Add unit, launch-gate, and E2E regression guards for OAuth queryParams.
