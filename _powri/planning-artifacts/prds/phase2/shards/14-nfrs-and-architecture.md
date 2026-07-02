<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §6–7: Cross-Cutting NFRs & Data Architecture

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §6–7 · **See also:** **Architecture:** [`architecture-phase2.md`](../../../architecture/phase2/architecture-phase2.md) · [`INDEX.md`](../../../INDEX.md)

---

## 6. Cross-Cutting NFRs

| ID | Requirement |
|----|-------------|
| **NFR-10** | All UGC sanitized on input; HTML stripped; DOMPurify on render. |
| **NFR-11** | Supabase Row Level Security: users read/write own trips/passport; public read on published reviews. |
| **NFR-12** | Photo upload: virus scan or MIME validation; max 5 photos/review, 1/check-in; 10 MB each; WebP/JPEG/PNG only. |
| **NFR-13** | Session idle timeout + absolute max; secure cookie flags. |
| **NFR-14** | Account deletion completes within 30 days; immediate session revocation on request. |
| **NFR-15** | Google Places + Supabase keys server-side only. |
| **NFR-16** | Core Web Vitals on resort detail with map: no regression vs Phase 1 (map lazy-loaded). |
| **NFR-17** | Adaptive nav: mobile bottom **Home · Explore · Saved · Plan**; desktop top nav adds Passport · Info · Account; hamburger overflow on mobile. |

---

## 7. Data Architecture Summary

PostHog is **analytics only** — not a user content database. Phase 2 requires:

| Layer | Technology | Purpose |
|-------|------------|---------|
| Database | Supabase PostgreSQL | Users, reviews, comments, trips, passport, saved resorts, reports |
| Auth | Supabase Auth | Google OAuth + email |
| File storage | Supabase Storage | Review + passport photos |
| Analytics | PostHog (EU) | Events, funnels — unchanged pattern |
| Resort content | Markdown (build-time) | Unchanged — no UGC in markdown |

Full schema: [`addendum.md`](addendum.md).

---
