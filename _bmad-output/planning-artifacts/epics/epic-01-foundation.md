# Epic 1: App Foundation & Content Pipeline

**Stories:** 1.1–1.7 · **Status:** Complete (see retro) · **Shard:** [`../epics.md`](../epics.md)

Establish Next.js app shell, design system, markdown pipeline, analytics/consent, bottom navigation.

**FRs:** Enables all · **NFRs:** NFR-3, 4, 5, 8, 11, 14 · **UX:** UX-DR1–3

---

## Story 1.1: Initialize Next.js Project & Deployment Baseline ✅

Next.js App Router, TypeScript, Tailwind, ESLint; `package-lock.json`; `npm audit` clean; HTTPS/HSTS; health-check route.

## Story 1.2: Design Tokens & Default Theme Typography ✅

Theme A CSS variables (UX-DR1); Cormorant Garamond + DM Sans (UX-DR2); 8px grid; radii per Theme A.

## Story 1.3: Markdown Content Loader & Resort Types ✅

gray-matter loader; `content_status: published` filter; TypeScript types for §8.1 fields.

## Story 1.4: App Shell, Routing & Bottom Navigation ✅

`BottomNav` Home · Explore · Info (UX-DR3); routes `/`, `/explore`, `/info`, `/resorts/[slug]`.

## Story 1.5: Analytics, Consent Banner & Common Event Properties ✅

PostHog EU + Plausible anonymous default; consent banner; shared `track()` with common properties.

## Story 1.6: Security Headers & API Route Scaffold ✅

Strict CSP; SRI; `/api/` namespace; rate-limit scaffold (10/min AI, 30/min search).

## Story 1.7: i18n Scaffold (English Only) ✅

next-intl or equivalent; EN only ships; `locale` on events defaults to `en`.

---

**Implementation artifacts:** [`../../implementation-artifacts/1-*.md`](../../implementation-artifacts/)  
**Retro:** [`../../implementation-artifacts/epic-1-retro-2026-06-13.md`](../../implementation-artifacts/epic-1-retro-2026-06-13.md)  
**Architecture:** [`../architecture.md`](../architecture.md)
