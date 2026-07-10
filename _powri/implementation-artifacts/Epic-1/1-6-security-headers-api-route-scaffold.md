# Story 1.6: Security Headers & API Route Scaffold

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **developer**,
I want CSP, CORS, and serverless route scaffolding in place,
So that later AI and search endpoints are secure by default.

## Acceptance Criteria

1. **Given** the production build  
   **When** security headers are inspected  
   **Then** strict CSP allows scripts only from trusted sources with no inline scripts/eval (NFR-3)

2. **Given** third-party scripts from CDN  
   **When** Plausible is configured  
   **Then** the script tag includes SRI (`integrity` + `crossOrigin`)

3. **Given** `/api/*` routes  
   **When** cross-origin requests arrive  
   **Then** CORS is restricted to the app's own domain

4. **Given** rate-limit policies from PRD §6.3  
   **When** middleware runs on protected routes  
   **Then** scaffold enforces 10 req/min (AI chat) and 30 req/min (search)

## Tasks / Subtasks

- [x] **CSP & security headers** (AC: 1)
  - [x] Extend `web/next.config.ts` with strict CSP (no `unsafe-eval`; no inline scripts)
  - [x] Whitelist PostHog EU + Plausible in `script-src` / `connect-src`
  - [x] Keep existing HSTS + `X-Content-Type-Options`; add `X-Frame-Options`, `Referrer-Policy`
  - [x] `style-src 'self' 'unsafe-inline'` OK (styles only — not scripts)

- [x] **Plausible SRI** (AC: 2)
  - [x] Add optional `NEXT_PUBLIC_PLAUSIBLE_SCRIPT_INTEGRITY` to `.env.example`
  - [x] Apply `integrity` + `crossOrigin="anonymous"` on Plausible `Script` when set

- [x] **API scaffold** (AC: 3, 4)
  - [x] `web/src/lib/api/errors.ts` — standard `{ error: { code, message } }` responses
  - [x] `web/src/lib/api/cors.ts` — origin validation helper
  - [x] `web/src/lib/rate-limit/` — in-memory limiter + policies (`aiChat`: 10/min, `search`: 30/min)
  - [x] `web/src/middleware.ts` — CORS gate + rate limits on `/api/chat`, `/api/search`; skip `/api/health`
  - [x] `web/src/app/api/chat/route.ts` — POST stub → 503 `AI_UNAVAILABLE`
  - [x] `web/src/app/api/search/route.ts` — POST stub → 501 `NOT_IMPLEMENTED`

- [x] **Validate** (AC: 1–4)
  - [x] `npm run build` + `npm run lint` pass
  - [x] `/api/health` still returns 200

- [x] **Out of scope**
  - Full AI proxy (Epic 4)
  - Upstash Redis rate limiter (document upgrade path in code comment)
  - DOMPurify input sanitization (Epic 4 chat)

## Dev Notes

### CSP template (next.config.ts)

```typescript
const csp = [
  "default-src 'self'",
  "script-src 'self' https://plausible.io",
  "connect-src 'self' https://eu.i.posthog.com https://*.i.posthog.com https://plausible.io",
  "img-src 'self' data: blob: https:",
  "style-src 'self' 'unsafe-inline'",
  "font-src 'self' data:",
  "frame-ancestors 'none'",
  "base-uri 'self'",
  "form-action 'self'",
  "object-src 'none'",
].join('; ');
```

PostHog SDK is bundled via npm (`'self'`). Plausible is external CDN.

### Rate limit error (architecture standard)

```json
{ "error": { "code": "RATE_LIMIT", "message": "Too many requests." } }
```

Status 429 with `Retry-After` header when limited.

### CORS rule

Same-origin only: if `Origin` header present, it must match request `Host`. No `Access-Control-Allow-Origin: *`.

### Previous story intelligence

- Story 1.1: HSTS already in next.config — extend, don't replace
- Story 1.5: Plausible via `next/script` — add SRI props
- `/api/health` must remain unblocked for deploy smoke tests

### References

- [Source: epics.md § Story 1.6](../planning-artifacts/epics/phase1/epics-phase1.md)
- [Source: _powri/planning-artifacts/prds/phase1/prd-phase1.md §6.2–6.3](../planning-artifacts/prds/phase1/prd-phase1.md)
- [Source: architecture.md § Authentication & Security](../planning-artifacts/architecture/phase1/architecture-phase1.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — pass; middleware active on `/api/*`
- `npm run lint` — pass

### Completion Notes List

- Strict CSP in `next.config.ts` (no unsafe-eval/inline scripts; PostHog + Plausible whitelisted)
- Added X-Frame-Options, Referrer-Policy, Permissions-Policy alongside HSTS
- Plausible Script supports SRI via `NEXT_PUBLIC_PLAUSIBLE_SCRIPT_INTEGRITY`
- Middleware: same-origin CORS gate; rate limits 10/min chat, 30/min search; health exempt
- API stubs: `/api/chat` (503), `/api/search` (501)

### File List

- `web/next.config.ts` (modified)
- `web/.env.example` (modified)
- `web/src/middleware.ts` (new)
- `web/src/lib/api/errors.ts` (new)
- `web/src/lib/api/cors.ts` (new)
- `web/src/lib/rate-limit/limiter.ts` (new)
- `web/src/lib/rate-limit/index.ts` (new)
- `web/src/app/api/chat/route.ts` (new)
- `web/src/app/api/search/route.ts` (new)
- `web/src/components/analytics/AnalyticsProvider.tsx` (modified — SRI)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Review Findings

- [x] [Review][Defer] In-memory rate limiter not multi-instance safe — documented; Upstash before scale
- [x] [Review][Defer] Plausible SRI hash must be set at deploy — env-driven, documented in `.env.example`
