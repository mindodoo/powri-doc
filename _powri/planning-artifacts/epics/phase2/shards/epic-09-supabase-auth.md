<!-- generated from epics/phase2/epics-phase2.md — do not edit; run prd-shard-regen.md -->

# Epic 9: Supabase Foundation & Auth

**Stories:** 9.1–9.5 · **Status:** Backlog · **Phase:** 2  
**FRs:** FR-8 · **NFRs:** NFR-10, 11, 13, 14, 15  
**Architecture:** [`architecture/phase2/architecture-phase2.md`](../../../architecture/phase2/architecture-phase2.md) §3–4  
**Full ACs:** [`epics-phase2.md`](../epics-phase2.md) — Epic 9

> **Gate:** Epic 9 must complete before Epics 12–14 (UGC/trips). Epic 10 can start after 9.2.

---

## Overview

| Story | Summary |
|-------|---------|
| **9.1** | Supabase project + migration 001 + Storage buckets + env vars |
| **9.2** | `@supabase/ssr` clients + middleware session refresh + `is_logged_in` analytics |
| **9.3** | SignInSheet (Google + magic link) + callback + PostHog alias + auth events |
| **9.4** | Profile username/display + `/account` settings + profanity filter |
| **9.5** | GDPR JSON export + account deletion request flow |

---

## Recommended order

```
9.1 → 9.2 → 9.3 → 9.4 → 9.5
```

**Dependencies:** none (first Phase 2 epic).  
**Blocks:** all logged-in features.

---

## Key paths

| Area | Path |
|------|------|
| Migration | `supabase/migrations/001_phase2_core.sql` |
| Supabase clients | `web/src/lib/supabase/` |
| Middleware | `web/src/middleware.ts` |
| Auth UI | `web/src/components/auth/SignInSheet.tsx` |
| Callback | `web/src/app/api/auth/callback/route.ts` |
| Account | `web/src/app/[locale]/account/page.tsx` |

---

## Validate (epic complete)

- [ ] Sign in Google + magic link E2E on preview
- [ ] Session persists refresh across navigation
- [ ] Export + deletion request flows work
- [ ] RLS: user A cannot read user B draft trips (smoke test)
