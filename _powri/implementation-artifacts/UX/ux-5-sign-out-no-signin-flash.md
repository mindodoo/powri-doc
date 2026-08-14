# Story UX-5: Sign-out with no sign-in flash

Status: ready-for-dev

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

- [ ] **Unit:** N/A unless extracting a `signOutToHome()` helper
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** Keep existing `auth_sign_out`; no new event required
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual: Account → Sign out — no Account guest CTA. Playwright: optional assert URL `/` without `/account` guest copy
- [ ] **Manual QA:** PO — blink gone

## Tasks / Subtasks

- [ ] Fix `handleSignOut` so Account cannot re-render as guest before navigation (AC: 1–2)
- [ ] Keep `window.location.href = '/'` (full load is OK) **or** navigate first then sign out — pick the one that cannot paint the guest Account block (AC: 1–3)
- [ ] Confirm `AccountDangerZone` delete path still goes Home (already `location.href = '/'`)

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

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
