# PRD §3.8: Phase 1 Improvements (Epic 6)

**Source:** Product owner feedback post–Epic 5 · **Epic shard:** [`../epics/epic-06-phase1-improvements.md`](../epics/epic-06-phase1-improvements.md)  
**Timing:** After Phase 1 build complete (Epics 1–3, 5); **before** manual launch QA gate.

---

## Purpose

Phase 1 shipped a complete editorial product. Epic 6 captures **incremental improvements** — UX refinements, content quality, and ops — without expanding scope into Phase 2 (maps, accounts) or Phase 3 (AI, monetisation).

---

## Functional requirements (improvements)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-1** | Home resort list sorted by **popularity rank** (not alphabetical); **5** cards default; **Show all → Explore** (no in-page expansion) | 6.1 |
| **IMP-2** | Home **quiz CTA** below hero: "Find your right ski resort" + button to `/quiz` | 6.2 |
| **IMP-3** | Quiz **#1 podium card**: rank **"1"** overlay on image; remove side "Your top match" column | 6.3 |
| **IMP-4** | Quiz **back arrow** → `/explore` on all quiz phases | 6.4 |
| **IMP-5** | Trail map section: **button-only** outbound link; note text has **no raw URLs** | 6.5 |

---

## Non-functional / quality (retro backlog)

| ID | Requirement | Story |
|----|-------------|-------|
| **IMP-6** | Highlight/lowlight columns: **16px icons** on column headers only (DESIGN.md polish) | 6.6 |
| **IMP-7** | **Unsplash images** for flagship resorts (Niseko, Hakuba, Nozawa minimum) | 6.7 |
| **IMP-8** | **≥3 daytrips** for GALA Yuzawa + Joetsu Kokusai | 6.8 |
| **IMP-9** | Verify/fix **broken external URLs** from launch link probe | 6.9 |
| **IMP-10** | **PostHog baseline dashboards** per tracking-plan §5 | 6.10 |
| **IMP-11** | **Production env documentation** (contact email, analytics keys) | 6.11 |
| **IMP-12** | Detail hero **quick verdict** smaller type + line clamp; Joetsu Kokusai name cleanup | 6.12 |

---

## Analytics changes (Epic 6)

| Event | Trigger | Phase | Story |
|-------|---------|-------|-------|
| `home_explore_cta_clicked` | Home "Show all resorts" → Explore | 1.1 | 6.1 |
| `quiz_cta_tapped` | Home hero quiz CTA (optional) | 1.1 | 6.2 |

**Deprecated behaviour:** Home in-page `load_more_clicked` — remove when 6.1 ships; Explore may retain load-more.

Update [`docs/tracking-plan.md`](../../../docs/tracking-plan.md) in the same PR as each story.

---

## Out of scope (Epic 6)

- User accounts, login, saved trips (Phase 2+)
- Resort **user reviews** section → Phase 3 [`phase-3-user-community.md`](../epics/phase-3-user-community.md)
- AI chat, monetisation (Phase 3)
- Full 20-resort photography (6.7 Phase B — optional follow-up)

---

## Release gate

Epic 6 is **recommended complete** before executing [`docs/launch-checklist.md`](../../../docs/launch-checklist.md) §3 manual QA. Minimum bar: **6.1–6.6 + 6.11**.
