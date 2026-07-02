<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.6: Phase 3 AI Entry Points (FR-13)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.6 · **See also:** **Epic:** [`epic-15-phase2-launch.md`](../../../epics/phase2/shards/epic-15-phase2-launch.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.6 Phase 3 AI Entry Points (FR-13)

**Description:** Visual stubs only — no live AI. Prepares user expectation and analytics baseline.

#### FR-13.1: Resort detail AI stub

**Guest or User** sees floating or inline **"Plan with AI"** affordance on resort detail (matches UX-DR17 placement from Phase 3 epic).

**Consequences:**
- Tap shows modal: "AI trip assistant coming soon — start a trip skeleton now" with CTA to FR-12 if logged in, or sign-in if not.
- `ai_entry_point_tapped` with `surface: resort_detail`, `ai_available: false`.

#### FR-13.2: Quiz results AI stub

**Guest or User** sees **"Want help planning?"** on quiz results podium view.

**Consequences:**
- Same modal behaviour; `surface: quiz_results`.
- Does not persist quiz result to profile.

---
