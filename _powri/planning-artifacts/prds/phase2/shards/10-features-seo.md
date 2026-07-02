<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.7: SEO & Content Access (FR-14)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.7 · **See also:** **Epic:** [`epic-15-phase2-launch.md`](../../../epics/phase2/shards/epic-15-phase2-launch.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.7 SEO & Content Access (FR-14)

**Description:** Powri's organic traffic depends on public resort content. Logged-in features must not hide editorial content.

#### FR-14.1: Public resort content

**Guest** can access all resort editorial sections (hero, highlights, lowlights, terrain data, Getting There, trail map link) without login.

**Consequences:**
- No `noindex` on resort pages.
- Aggregate rating + review count visible to all users; full review bodies per FR-14.3.
- Trips and passport editing require login; trip content not indexed.
- Server-rendered resort pages remain SSG/ISR — editorial body in static HTML; UGC per FR-14.3.

#### FR-14.2: Auth-gated surfaces only

Login required **only** for: create/edit review, create/edit comment, report, trip CRUD, passport check-in, profile edit, cross-device saved-resort sync.

#### FR-14.3: SEO strategy for UGC (OQ-1 resolved)

**System** preserves crawlable editorial body while limiting SEO risk from thin/duplicate UGC pages.

**Consequences:**
- Resort detail **editorial sections** remain fully SSR/SSG in initial HTML (hero, highlights, Getting There, etc.).
- **Aggregate** rating + review count rendered in SSR HTML for rich-snippet eligibility.
- **Individual review text** loaded client-side after hydration (not duplicated in static HTML per review URL — no per-review routes in Phase 2).
- No `noindex` on resort pages.
- Public profile pages (`/u/:username`) use `noindex, follow` until review volume justifies index (revisit Phase 2.1).
- Trips remain private — never indexed.

---
