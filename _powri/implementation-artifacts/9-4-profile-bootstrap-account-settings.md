# Story 9.4: Profile Bootstrap & Account Settings

Status: done

## Story

As a **signed-in user**,
I want a profile with display name and profile photo and an account settings page,
So that my public identity appears on reviews and comments.

## Acceptance Criteria

1. `/account` — edit username, display name, bio, profile photo (upload only)
2. Uploaded profile photo auto-saves to `profiles.avatar_url` and displays in the profile photo preview immediately; without a photo, display name initial is shown
3. Returning to `/account` shows the saved photo (and other profile fields)
4. Upload validation: JPG/PNG/WebP, max 2 MB; errors shown inline under upload controls (not buried at form bottom)
5. Display name profanity filter server-side
6. Username prompt when default `user-{id}` pattern on gated actions
7. Admin role not self-service (omit `role` from PATCH)

## Testing & Definition of Done

- [x] Unit: username validation, profanity filter, avatar URL validation
- [x] `lint`, `build`, `test:unit`, `test:analytics`, `test:launch`
- [x] Manual E2E: upload photo on `/account` — preview updates, persists after reload; oversize file shows inline error under upload
- [ ] Manual E2E: username prompt after gated sign-in with default username

## Tasks / Subtasks

- [x] `GET` + `PATCH /api/account/profile` (auth required; `role` omitted from PATCH)
- [x] Server-side display name profanity filter (`web/src/lib/ugc/profanity.ts`)
- [x] `/account` page + `AccountSettingsForm` + `AvatarPicker` (Supabase Storage upload only; noindex metadata)
- [x] Profile photo auto-save on upload/remove (`PATCH avatar_url`); data-URL preview during upload; cache-bust query on stored upload URLs
- [x] Client upload validation (`validateAvatarFile`) + photo errors under upload button; form errors separate above Save
- [x] Theme token `--color-error` + semantic `.text-error` in `globals.css` for validation feedback
- [x] `supabase/migrations/003_avatars_bucket.sql`; `verify:supabase` checks `avatars` bucket
- [x] `AuthProvider`: `profile`, `refreshProfile`, `requireAuthForAction`, username prompt sheet
- [x] Nav **Account** link when signed in
- [x] i18n strings in `messages/en.json`

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Default username pattern: `/^user-[a-f0-9]{12}$/i` (matches DB bootstrap trigger).
- Gated-action trigger persisted in `sessionStorage` (`powri_auth_trigger`) so OAuth redirect survives before username prompt.
- Profile photo: user upload to Supabase `avatars` bucket (`003_avatars_bucket.sql`); auto-saved on change; cache-bust query on upload URLs; max 2 MB JPG/PNG/WebP. No preset avatars — initials fallback when no photo.
- Upload errors render under **Upload photo** via `avatarErrorMessage`; uses `.text-error` → `--color-error` (unlayered semantic utility in `globals.css`, not Tailwind-only).
- **Ops:** `npm run verify:supabase` must include `avatars` bucket — apply `supabase/migrations/003_avatars_bucket.sql` via `npx supabase db push` or SQL Editor if missing.
- `AccountPageClient` uses `key={profile.id}` (not `updated_at`) so photo save does not remount the whole form mid-edit.

### File List

- `supabase/migrations/003_avatars_bucket.sql`
- `scripts/verify-supabase-schema.mjs`
- `web/src/app/api/account/profile/route.ts`
- `web/src/app/[locale]/account/page.tsx`
- `web/src/components/account/{AccountPageClient,AccountSettingsForm,AvatarPicker}.tsx`
- `web/src/components/auth/{AuthProvider,UsernamePromptSheet}.tsx`
- `web/src/components/layout/AppMenuSheet.tsx`
- `web/src/lib/api/errors.ts`
- `web/src/lib/auth/{avatarUrl,uploadAvatar,profileTypes,username,requireUser,profileClient}.ts` + `{avatarUrl,username}.test.ts`
- `web/src/lib/ugc/profanity.ts` + `profanity.test.ts`
- `web/src/app/globals.css`
- `web/src/styles/tokens.css`
- `web/messages/en.json`

## Senior Developer Review (AI)

**Review date:** 2026-07-02  
**Outcome:** Approve (with post-launch avatar upload hardening 2026-07-02; preset avatars removed 2026-07-02)

- [x] All ACs implemented; PATCH rejects profanity and duplicate usernames
- [x] `role` not exposed in client PATCH body
- [x] Username prompt wired via `requireAuthForAction` for Epics 12–14
- [x] Profile fetch uses authenticated Supabase server client
- [x] Profile photo upload auto-save; inline validation UX; initials fallback; `avatars` bucket migration documented
