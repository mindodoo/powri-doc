# PRD Quality Review — Japan Ski & Snowboard Trip Planner

**PRD:** `_powri/planning-artifacts/prds/phase1/prd-phase1.md` (v1.0 Draft, ~1,333 lines)  
**Validated:** 2026-06-12  
**Stakes:** Launch-oriented solo + AI build (not hobby; not enterprise)

## Overall verdict

This is an **above-average brownfield PRD** for a consumer travel product — unusually thorough on content strategy, analytics, legal, GTM, AI cost guardrails, and release gates. A solo builder could start Phase 1 with confidence on *what* to build and *why*.

The main risks are **internal contradictions** (scope vs screens vs analytics vs content count), **unresolved architecture forks** (CMS vs markdown, maps stack, design theme), and **missing downstream primitives** (FR IDs, user journeys, counter-metrics, satisfaction instrumentation). None are fatal, but several would cause rework if epics are written before reconciliation.

**Grade: Good** — strong substance, a handful of high-severity consistency gaps to fix before epics.

---

## Decision-readiness — adequate

Decisions in §11 are clear and actionable (monetisation, photography, accounts, languages, data accuracy). Release gates (§10.2) are well-defined for a solo+AI workflow.

### Findings
- **high** Scope fork unresolved (§3.1 vs §10.1 vs §8) — §3.1 promises 15–20 destinations and lists all 20; Phase 1 ships **top 10 only**; content prep targets all 20. *Fix:* Add explicit Phase 1 resort list (which 10) and mark the other 10 as content-ready-but-not-shipped or hidden.
- **high** Content pipeline fork (§7.3 vs §8) — Architecture recommends Contentful/Sanity CMS; content actually lives in `content/resorts/*.md` git files. *Fix:* Decide: (A) markdown → build-time import, (B) CMS as source of truth, or (C) hybrid — and state it as a resolved decision in §11.
- **medium** Maps stack undecided (§7.1) — "Leaflet + OSM (low cost) **or** Google Maps." Risk register R2 assumes OSM default but FRs don't commit. *Fix:* Pick default for Phase 1; defer Google to Phase 2+ if needed.
- **medium** Design theme undecided (§12.2–12.3) — Four reference themes documented; "Default Theme" uses Cormorant/DM Sans but §5.1 still lists Playfair/DM Sans; Theme B (Noto) flagged for JP i18n. *Fix:* One resolved V1 theme + note when Theme B activates (with TH/JP launch).

---

## Substance over theater — strong

Content volatility tiers, season currency policy, analytics discipline, and honest lowlights are earned content — not template filler. GTM rings (§15) and risk register (§16) are product-specific.

### Findings
- **low** Target users (§1.3) are persona-like but don't drive decisions — acceptable given §12 screen specs carry the UX weight instead of formal UJs.

---

## Strategic coherence — adequate

Clear thesis: trustworthy, magazine-quality Japan ski planning (not a booking engine). Phases follow dependency order. Success metrics align with discovery → detail → accommodation intent funnel.

### Findings
- **high** No counter-metrics (§2) — e.g. bounce rate on home, time-on-site without detail view, filter-to-zero-results rate. Without these, optimising toward ≥70% detail engagement could hide UX regressions. *Fix:* Add 2–3 guardrail metrics tied to §13 dashboard.
- **medium** Satisfaction ≥4.2/5 (§2) has no collection mechanism — no prompt timing, sample rate, or question wording. *Fix:* Specify in-app micro-survey (e.g. after 2nd resort detail view) or defer metric to Phase 2 with explicit note.

---

## Done-ness clarity — thin

Features in §3 are rich in description but lack testable acceptance criteria and stable IDs. §10.2 gates reference "acceptance criteria" but none are enumerated per feature.

### Findings
- **high** No FR/UJ/SM IDs — downstream epics will invent their own numbering; cross-refs will drift. *Fix:* Add lightweight IDs to §3 features (FR-1, FR-2…) with 1–3 acceptance bullets each for Phase 1 scope only.
- **medium** Hero video loop (§3.2 §2a) conflicts with Unsplash-only photography (§11.2) — "video loop background on Wi-Fi" has no asset source or phase. *Fix:* Mark as Phase 4 or out of scope; V1 = static hero carousel only.
- **medium** Terrain map (§3.2 §2c) mentions Fatmap/Slopes embed; content model uses official `trail_map_url` link-out. *Fix:* Clarify V1 = link to official map (already in content schema); embed is future.

---

## Scope honesty — adequate

§9 Out of Scope exists. §10 explicitly avoids calendar fiction. Many deferrals are honest (accounts Phase 2, AI Phase 3).

### Findings
- **critical** Navigation / feature phase mismatch — §5.3 bottom nav includes **My Trip** (saved itinerary); §12.5 uses **Plan** tab; §11.4 defers accounts/saved trips to Phase 2; AI chat is Phase 3 but onboarding Screen 0 card 4 and §12 Screen 5 describe AI. *Fix:* Define Phase 1 bottom nav explicitly (e.g. Home · Explore · Info only, or Plan = static checklist stub) and tag §12 screens with phase labels.
- **high** Quiz scope mismatch — §13.3 instruments `quiz_started`/`quiz_completed` at launch; §10.1 Phase 3 lists onboarding quiz. §12 Screen 0b fully specifies quiz UX. *Fix:* Either move quiz to Phase 1 (it's lightweight) or remove from Phase 1 analytics/onboarding and gate Screen 0b to Phase 3.
- **medium** §9 says "User accounts and social features (V2)" but §11.4 says Phase 2 — minor version label inconsistency. *Fix:* Align to "Phase 2" everywhere.

---

## Downstream usability — adequate

§8 file-map is excellent for AI agents and implementers. Content schema (§8.1) mirrors content-model.md. Tracking plan is externalized.

### Findings
- **high** No glossary — terms like T1/T2/T3, resort slug, gateway city, valley-pass used across §4, §8, §13 without a single definition block. *Fix:* Short glossary appendix or pointer that content-model.md §2 is canonical for content tiers.
- **medium** No user journeys with named protagonists — for a consumer UX-heavy product, 2–3 short journeys (e.g. "Sarah, first Japan trip, intermediate skier") would de-risk epic slicing. Optional before epics if §12 screens are treated as source of truth.
- **low** Resort #16 named "Nendai (Joetsu Kokusai)" in seed table vs `joetsu-kokusai.md` slug — mechanical naming drift.

---

## Shape fit — strong

Shape matches a solo+AI consumer product: heavy on content/legal/analytics, light on enterprise process. Release gates replace calendar milestones appropriately.

### Findings
- **low** PRD length (~1,333 lines) is justified for launch stakes but §5 UI/UX Design Requirements partially duplicates §12 — consider marking §5 as superseded by §12 to reduce dual-maintenance.

---

## Mechanical notes

- **Status:** Still `Draft` in header — should move to `final` (or `ready-for-epics`) after gap fixes.
- **Cross-ref integrity:** §8, §11, §13, §17 cross-reference each other well; §5 ↔ §12 typography/nav conflicts need reconciliation.
- **No Open Questions / Assumptions index** — deferrals are embedded in prose; a single "Open items" table at end would help gate reviews.
- **Brownfield label** in project-context.md is misleading — app code doesn't exist; content prep is brownfield, product is greenfield.
