---
baseline_commit: 5022fdea8d453e7e4361d596bb9ba77a1150d797
---

# Bug 9.8: Magic link callback — `auth_error=1` redirect; account created but session not established

Status: done  
**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`)  
**Severity:** P1 — magic link sign-up partially succeeds; user is blocked after first attempt  
**Related:** Bug 9.6/9.7 (prod URL config) · Bug 9.9 (error copy; delivery still rate-limited on built-in SMTP)

---

## Symptom

On production, a user who signs up via email magic link:

1. Receives the magic link email (when Supabase send quota allows)
2. Clicks the link → redirected back to Powri with **`?auth_error=1`** in the URL
3. User is **not signed in** — appears as a failed sign-up
4. However, the **Supabase account was already created** for that email address
5. User may be stuck: account exists but session was never established; retry depends on magic-link delivery (Bug 9.9 / custom SMTP)

**Reproducibility (2026-07-03, PO debug):** Fails on **every** magic-link sign-in attempt on production after ops fixes below — not intermittent.

---

## Investigation timeline

### Phase 1 — before PKCE forced on server route

| Observation | Interpretation |
|-------------|----------------|
| Callback hit `if (!code)` branch → `auth_error=1` | Supabase returned **implicit-flow** tokens (access token in URL **hash**), not a PKCE `?code=` query param |
| Server callback only handles PKCE `code` | **Flow mismatch** between magic-link initiation and callback handler |

Ops change attempted: set Supabase **Site URL** to `https://powri.vercel.app` and add `NEXT_PUBLIC_SITE_URL` on Vercel production. This fixed redirect URL generation (related to 9.6/9.7) but did not fix session establishment.

### Phase 2 — after `flowType: 'pkce'` on `/api/auth/magic-link`

| Observation | Interpretation |
|-------------|----------------|
| Callback now receives `?code=` correctly | PKCE redirect from Supabase works |
| Server log: `AuthPKCECodeVerifierMissingError: PKCE code verifier not found in storage` | **PKCE verifier never persisted to browser cookies** during magic-link send |

Initial story hypothesis (Chrome cross-site cookie blocking when opening link from Gmail) is **ruled out** for the reproducible case. The verifier cookie was **never set**, not blocked on redirect.

---

## Root cause (confirmed by code trace)

`?auth_error=1` is set by `/api/auth/callback/route.ts` when:

1. The callback is hit **without** a `code` query param, or
2. `supabase.auth.exchangeCodeForSession(code)` returns an error.

### Architectural mismatch: Google OAuth vs magic link

| Step | Google OAuth (works) | Magic link (broken) |
|------|----------------------|---------------------|
| Initiation | Browser `createBrowserClient()` in `SignInSheet.tsx` | Server `/api/auth/magic-link` with bare `@supabase/supabase-js` |
| Client library | `@supabase/ssr` — cookie-based PKCE storage | `@supabase/supabase-js` with `persistSession: false`, no cookie adapter |
| PKCE verifier | Written to `sb-*-auth-token-code-verifier` cookie in browser | Generated in server memory during `signInWithOtp`, **discarded when POST completes** |
| Callback | Reads verifier from cookies → `exchangeCodeForSession` succeeds | No verifier in cookies → `AuthPKCECodeVerifierMissingError` |

Google OAuth path (reference):

```ts
// SignInSheet.tsx — browser @supabase/ssr client
const supabase = createClient();
await supabase.auth.signInWithOAuth({ provider: 'google', options: { redirectTo } });
```

Broken magic-link path:

```ts
// /api/auth/magic-link/route.ts — ephemeral server client
const supabase = createClient(getSupabaseUrl(), getSupabaseAnonKey(), {
  auth: { flowType: 'pkce', autoRefreshToken: false, persistSession: false },
});
await supabase.auth.signInWithOtp({ email, options: { emailRedirectTo } });
```

Callback (expects verifier in cookies from initiation):

```ts
// /api/auth/callback/route.ts
const supabase = await createClient(); // @supabase/ssr server client
await supabase.auth.exchangeCodeForSession(code);
```

Supabase docs: PKCE magic links require the code verifier in **shared cookie storage** between initiation and callback. Server-side initiation must use `@supabase/ssr` with cookies propagated to the browser, **or** initiation must happen in the browser (same pattern as OAuth).

### Why account exists but session does not

| Step | What happens |
|------|-------------|
| 1 | `signInWithOtp` sends the email (verifier not persisted to browser) |
| 2 | User clicks link → Supabase validates OTP, creates account if new, redirects with `?code=` |
| 3 | Callback calls `exchangeCodeForSession(code)` → fails without verifier → `auth_error=1` |

---

## Fix plan

### 1. Move magic-link initiation to browser client (primary fix)

In `SignInSheet.tsx`, call `signInWithOtp` via `createClient()` (browser `@supabase/ssr`), matching Google OAuth:

- Use `buildAuthCallbackUrl(window.location.origin, returnTo)` for `emailRedirectTo`
- Map Supabase errors to existing i18n keys (`magicLinkSendFailed`, `magicLinkRateLimited`, etc.) — reuse logic from `magicLinkErrors.ts` where applicable

This ensures the PKCE verifier cookie is set **before** the user leaves for their inbox.

### 2. Remove or retire `/api/auth/magic-link` route

**Recommended (Option A):** Delete `web/src/app/api/auth/magic-link/route.ts` and remove its rate-limit entry from `middleware.ts`. Rationale: anon key is already public; Supabase enforces send rate limits server-side.

**Alternative (Option B):** Keep route as validation-only proxy without calling Supabase — only if server-side rate limiting on send is required. Not recommended for minimal diff.

### 3. Harden callback cookie handling (secondary)

Update `/api/auth/callback/route.ts` to bind `@supabase/ssr` cookie `setAll` to the redirect `NextResponse` explicitly (Supabase recommended pattern for route handlers). Belt-and-suspenders after primary fix.

Remove temporary debug `console.log` calls once verified; **keep** structured error logging for `exchangeCodeForSession` failures (AC3).

### 4. User-friendly recovery for `auth_error=1` (unchanged — still required)

Regardless of root cause fix, surface failures in UI:

- In `AuthProvider`, detect `?auth_error=1` on mount
- Show readable copy: link expired or already used; offer CTA to reopen `SignInSheet`
- Strip `auth_error` from URL via `history.replaceState` (same pattern as `auth=signed_in`)
- Add i18n strings in `web/messages/en.json`

---

## Tasks / Subtasks

- [x] **Task 1:** Move `signInWithOtp` to browser client in `SignInSheet.tsx`; preserve analytics (`trackMagicLinkSent`) and error mapping
- [x] **Task 2:** Remove `/api/auth/magic-link` route; remove `/api/auth/magic-link` from auth rate-limit list in `middleware.ts`
- [x] **Task 3:** (Optional) Refactor callback route to bind cookies to redirect response
- [x] **Task 4:** Add `auth_error` handler in `AuthProvider` + i18n copy
- [x] **Task 5:** Remove debug logging from callback; retain error-code logging without PII
- [x] **Task 6:** Manual E2E on production (or staging with SMTP): fresh magic link → signed in, `?auth=signed_in`, no `auth_error=1`
- [x] **Task 7:** Run DoD from `web/`: `lint`, `build`, `test:unit` (if auth lib touched), `test:analytics` (if analytics touched)

---

## Acceptance criteria

1. [x] Clicking a fresh magic link signs the user in and redirects to intended `returnTo` with `?auth=signed_in` — no `?auth_error=1` *(pending Task 6 manual E2E)*
2. [x] If the callback still fails, the user sees a user-friendly error message with a "Request new link" CTA — not a raw `?auth_error=1` param with no explanation
3. [x] Server-side logs capture the `exchangeCodeForSession` error code without PII for future diagnostics
4. [x] An orphaned account (account exists, no session) can successfully sign in on a retry after the error is surfaced *(pending Task 6 manual E2E)*

---

## Testing & Definition of Done

- [x] Unit: existing `returnTo`, `magicLinkErrors` tests still pass; add/adjust tests if client-side error mapping changes
- [x] `npm run lint && npm run build && npm run test:launch` from `web/` *(test:launch warns on content stamps + missing CONTACT_EMAIL locally; no auth regressions)*
- [x] `npm run test:unit` if `src/lib/auth/**` touched
- [x] `npm run test:analytics` if auth analytics paths touched
- [x] Manual E2E: magic link send → inbox → click → session established (production or env with working SMTP)

**Note:** Production magic-link **send** may still hit Supabase built-in 2 emails/hour limit (Bug 9.9 scope). Callback fix can be verified on staging/local with working email, or production when SMTP/quota allows.

---

## Files involved

| File | Role |
|------|------|
| `web/src/components/auth/SignInSheet.tsx` | **Primary fix** — move `signInWithOtp` to browser client |
| `web/src/app/api/auth/magic-link/route.ts` | **Remove** — server initiation causes missing verifier |
| `web/src/middleware.ts` | Remove magic-link from auth rate-limit paths if route deleted |
| `web/src/app/api/auth/callback/route.ts` | Session exchange; error logging; optional cookie hardening |
| `web/src/components/auth/AuthProvider.tsx` | `?auth_error=1` user-facing handler |
| `web/src/lib/auth/magicLinkErrors.ts` | Reuse error mapping on client if needed |
| `web/messages/en.json` | Auth error recovery copy |

---

## Dev Agent Record

### Agent Model Used

Composer (investigation + story update, 2026-07-03) · Composer (implementation, 2026-07-03)

### Completion Notes

**Root fix:** Moved `signInWithOtp` from ephemeral server route to browser `@supabase/ssr` client in `SignInSheet.tsx`, matching Google OAuth. PKCE verifier cookie is now set in the browser before the user opens their inbox.

**Removed:** `/api/auth/magic-link` route and its middleware rate-limit entry.

**Callback hardening:** Refactored `/api/auth/callback` to collect session cookies during `exchangeCodeForSession` and apply them to the redirect `NextResponse`. Replaced debug `console.log` with structured `[auth/callback]` error logs (`missing_code`, `exchange_failed` + error code/name, no PII).

**Recovery UX:** `AuthProvider` detects `?auth_error=1`, opens `SignInSheet` with recovery copy and strips the param from the URL.

**Automated checks:** `lint`, `build`, `test:unit` (106 pass), `test:analytics`, `test:launch` (pre-existing local CONTACT_EMAIL warning only).

**PO sign-off (2026-07-08):** Manual E2E verified — fresh magic link → `?auth=signed_in` on staging; orphaned-account retry succeeds.

### File List

- `web/src/components/auth/SignInSheet.tsx` — browser `signInWithOtp`; auth callback error banner
- `web/src/app/api/auth/magic-link/route.ts` — **deleted**
- `web/src/middleware.ts` — removed magic-link from auth rate-limit paths
- `web/src/app/api/auth/callback/route.ts` — cookie-bound redirect; structured error logging
- `web/src/components/auth/AuthProvider.tsx` — `auth_error=1` handler
- `web/messages/en.json` — auth callback error recovery copy
- `_powri/implementation-artifacts/sprint-status.yaml` — status tracking

### Debug context (PO, 2026-07-03)

- Forced `flowType: 'pkce'` on server magic-link route to diagnose implicit vs PKCE mismatch
- Set `NEXT_PUBLIC_SITE_URL=https://powri.vercel.app` on Vercel production
- Added callback logging; confirmed `AuthPKCECodeVerifierMissingError` on every attempt
- Implementation deferred pending story approval
