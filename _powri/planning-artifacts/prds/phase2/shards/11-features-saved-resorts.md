<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.8: Saved Resorts (FR-15)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.8 · **See also:** **Epic:** [`epic-10-nav-saved.md`](../../../epics/phase2/shards/epic-10-nav-saved.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.8 Saved Resorts (FR-15)

**Description:** Users save resorts with a **heart** icon from resort cards and detail pages. Saved list is a first-class **Saved** destination in navigation. Feeds Phase 3 AI context (`saved_resorts` in trip handoff payload). Guests may save on-device; login merges local saves to account.

Realizes UJ-1 (resolution — Yuki hearts Niseko for later).

#### FR-15.1: Save / unsave resort

**Guest or User** can tap heart on resort card or detail page to save or unsave.

**Consequences:**
- **Guest:** saves persist in `localStorage` on current device only.
- **User:** saves persist in PostgreSQL; sync across devices.
- On first login after guest saves: merge localStorage slugs into account (dedupe); clear local copy after merge.
- Heart animates with subtle fill; active state visible on card and detail.
- `resort_saved` / `resort_unsaved` analytics with `resort_slug`, `is_logged_in`.

#### FR-15.2: Saved list view

**Guest or User** can open **Saved** from nav and see all saved resorts as resort cards (same card component as Explore).

**Consequences:**
- Route: `/saved` — public route, no login wall; empty state CTA to Explore.
- Guest sees banner: "Sign in to sync saved resorts across devices" (dismissible).
- Sort: most recently saved first.

**Out of Scope Phase 2:**
- Saved search queries or filters.
- Sharing saved list with others.

---
