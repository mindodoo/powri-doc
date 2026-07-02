# Story 9.2: Supabase SSR Clients & Session Middleware

Status: done

## Story

As a **returning user**,
I want my session to persist securely across page loads,
So that I stay signed in without storing tokens in localStorage.

## Acceptance Criteria

1. **Given** `@supabase/supabase-js` and `@supabase/ssr` are installed  
   **When** I navigate the app  
   **Then** `lib/supabase/client.ts`, `server.ts`, and `middleware.ts` exist per architecture-phase2 §4.2

2. **And** middleware calls session refresh before intl routing (after `/ingest` bypass)

3. **And** JWT is stored in HTTP-only cookies only (`@supabase/ssr` — not `localStorage`)

4. **And** `getCommonProperties()` sets `is_logged_in: true` when session exists

## Testing & Definition of Done

- [x] **Unit:** `commonProperties.test.ts`, `env.test.ts`
- [x] **Analytics:** `npm run test:analytics`
- [x] **CI placeholders:** Supabase env vars in `.github/workflows/ci.yml`
- [x] **PR gates:** `lint`, `build`, `test:unit`, `test:launch`, `test:analytics` pass
- [x] **Manual:** N/A until Story 9.3 sign-in UI

## Tasks / Subtasks

- [x] Install `@supabase/supabase-js` + `@supabase/ssr` in `web/`
- [x] Add `web/src/lib/supabase/{env,client,server,middleware,admin}.ts`
- [x] Wire `updateSession()` in `web/src/middleware.ts` before intl (after `/ingest`)
- [x] Client session sync → `getCommonProperties().is_logged_in`
- [x] Add `AuthSessionSync` in root layout
- [x] CI env placeholders for Supabase

## Dev Notes

### Session → analytics

`AuthSessionSync` (client) listens to `onAuthStateChange` and updates `sessionState.ts`; `getCommonProperties()` reads synchronously at event capture time.

### Middleware order

```
/ingest → bypass
/api/*  → CORS + rate limit (session refresh deferred)
pages   → updateSession → CSP → intl → merge session cookies
```

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes List

- `@supabase/ssr` `createBrowserClient` / `createServerClient` use cookies (not localStorage).
- `admin.ts` added per architecture §4.2; throws if service role key missing (server-only).
- Coverage gate preserved with `env.test.ts`.

### File List

- `web/package.json`, `web/package-lock.json`
- `web/src/lib/supabase/{env,client,server,middleware,admin}.ts`
- `web/src/lib/supabase/env.test.ts`
- `web/src/lib/auth/sessionState.ts`
- `web/src/components/auth/AuthSessionSync.tsx`
- `web/src/lib/analytics/commonProperties.ts`
- `web/src/lib/analytics/commonProperties.test.ts`
- `web/src/middleware.ts`
- `web/src/app/layout.tsx`
- `.github/workflows/ci.yml`
- `docs/qa/phase2/deploy-environment.md`
- `_powri/implementation-artifacts/9-2-supabase-ssr-clients-session-middleware.md`
- `_powri/implementation-artifacts/sprint-status.yaml`

## Senior Developer Review (AI)

**Review date:** 2026-07-02  
**Outcome:** Approve

### Summary

Correct Supabase SSR layering: middleware refreshes session before intl; cookies merged onto final response. Analytics `is_logged_in` wired via auth state subscription without making `track()` async.

### Findings

- [x] [Pass] All 4 ACs satisfied
- [x] [Pass] No JWT in localStorage
- [x] [Pass] CI placeholders prevent build failures
- [x] [Note] API routes skip session refresh — acceptable per AC; add in 9.3 if auth callback needs it
