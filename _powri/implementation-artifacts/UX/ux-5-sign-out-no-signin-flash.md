---
baseline_commit: 90c462847526fa2cff517054b8356a97b4b7b7d7
---

# Story UX-5: Sign-out with no sign-in flash

Status: done

Version when done: **2.12.2.5**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a signed-in user who signs out,  
I want to land on Home immediately,  
so that I never see a blink of a sign-in screen I cannot use.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-5).

## Acceptance Criteria

1. After **Sign out**, the next painted destination is **Home** (`/`).
2. **No flash** of Account’s unauthenticated “sign in required” CTA, SignInSheet, or any dedicated sign-in route.
3. There is **no** usable sign-in page on this path — do not add a `/sign-in` screen.
4. Deletion-pending sign-out (`AuthProvider` `auth=deletion_pending`) may still show notices as designed; this story is the **Account Sign out** control.
5. `auth_sign_out` still fires if it already does.

## Testing & Definition of Done

- [x] **Unit:** `web/src/lib/auth/signOutToHome.test.ts` — guest CTA predicate + `signOutThenGoHome` order
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** `auth_sign_out` remains in the registry; it was not fired in app code before this story and is not newly wired
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** Guest CTA gated by `isSigningOut`; `location.replace('/')` after sign-out. Playwright optional — not added (needs live session)
- [x] **Manual QA:** PO — blink gone

## Tasks / Subtasks

- [x] Fix `handleSignOut` so Account cannot re-render as guest before navigation (AC: 1–2)
- [x] Keep `window.location.href = '/'` (full load is OK) **or** navigate first then sign out — pick the one that cannot paint the guest Account block (AC: 1–3)
- [x] Confirm `AccountDangerZone` delete path still goes Home (already `location.href = '/'`)

### Review Findings

- [x] [Review][Patch] Delete path should reuse `signOutThenGoHome` for consistent navigate-on-failure [`AccountDangerZone.tsx:59-66`]
- [x] [Review][Defer] `auth_sign_out` never instrumented in app code — AC5 satisfied vacuously; registry `instrumentationFiles: []` pre-dates this story [`phase2Events.ts:33-35`] — deferred, pre-existing analytics gap

## Dev Notes

**Root cause (current code):** `AccountSettingsForm.handleSignOut` awaits `supabase.auth.signOut()` then `window.location.href = '/'`. Auth context clears **before** navigation → `AccountPageClient` hits `isAuthReady && !isAuthenticated` and paints the **Sign in** CTA for a frame. That is the “hidden sign-in page” blink — not a separate route. `middleware.ts` does **not** gate `/account`.

**Fix pattern:** Set a local `isSigningOut` that AccountPageClient already has via form… actually `isSigningOut` is inside the form; parent still shows the form until unmount. Better: `window.location.replace('/')` **before** awaiting signOut, or keep rendering the authenticated form until unload, or `router.replace('/')` with a blocking overlay and skip the guest branch when `isSigningOut`.

**Do not:** `openSignIn()` on sign-out. Redirect to `/auth/*`. Change Google OAuth callback.

### Files

| Path | Role |
|------|------|
| `web/src/components/account/AccountSettingsForm.tsx` | `handleSignOut` |
| `web/src/components/account/AccountPageClient.tsx` | Do not paint guest CTA during sign-out |
| `web/src/components/auth/AuthProvider.tsx` | Do not open SignInSheet on normal sign-out |

### References

- PRD UX-5
- Epic 9 auth (session is cookie + client)

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 1 pre-existing warning (`supabaseStorage.ts` unused `contentType`)
- `npm run build`: pass
- `npm run test:unit`: 98 files, 469 tests pass
- `npm run test:launch`: fails on missing `NEXT_PUBLIC_CONTACT_EMAIL` (pre-existing env; not caused by this change)
- `AuthProvider`: `SIGNED_OUT` does not call `openSignIn`; SignInSheet only for user CTA, `auth_error`, and `deletion_pending` — no code change

### Completion Notes List

- 2026-08-21: Lifted `isSigningOut` to `AccountPageClient` via `onLeavingAccount` so the guest “sign in required” CTA cannot paint after `signOut()` clears the session. Navigation uses `window.location.replace('/')` after sign-out (full load). Delete-account path also sets the leave flag and replace-Home. No `/sign-in` route. `auth_sign_out` was not previously instrumented; left unchanged.
- 2026-08-21: Code review patch applied — `AccountDangerZone` delete path uses `signOutThenGoHome`.

### File List

- `_powri/implementation-artifacts/UX/ux-5-sign-out-no-signin-flash.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/lib/auth/signOutToHome.ts`
- `web/src/lib/auth/signOutToHome.test.ts`
- `web/src/components/account/AccountPageClient.tsx`
- `web/src/components/account/AccountSettingsForm.tsx`
- `web/src/components/account/AccountDangerZone.tsx`

### Change Log

- 2026-08-21: Implemented sign-out → Home with no Account guest CTA flash (`isSigningOut` gate + `location.replace`).
- 2026-08-21: Code review patch — delete path uses `signOutThenGoHome` (same finally-navigate as sign-out).
- 2026-08-21: Code review complete; story marked done.

### Review Findings (summary)

**Outcome: Approve** — AC 1–4 implemented; AC 5 vacuously met. Core fix (`isSigningOut` gate + `signOutThenGoHome`) correctly prevents guest CTA flash.

**Dismissed (noise / out of scope):** dual `isSigningOut` state in form+parent (works); optional Playwright omitted per story; `AuthProvider` unchanged by design; `onLeavingAccount` prop drill acceptable; untracked helper files are commit hygiene not logic defects.

### Senior Developer Review (AI)

**Review date:** 2026-08-21  
**Outcome:** Approve

#### Action Items

- [x] [Review][Patch] Delete path should reuse `signOutThenGoHome` [`AccountDangerZone.tsx:59-66`]
- [x] [Review][Defer] `auth_sign_out` not wired — pre-existing; AC5 literal pass [`phase2Events.ts:33-35`]
