# PRD & epic shard regeneration

**When to run:** After any edit to a **monolith** (source of truth):

| Monolith (edit here) | Regenerate |
|----------------------|------------|
| [`prds/phase1/prd-phase1.md`](prds/phase1/prd-phase1.md) | [`prds/phase1/shards/`](prds/phase1/shards/) (01–18) |
| [`prds/phase2/prd-phase2.md`](prds/phase2/prd-phase2.md) | [`prds/phase2/shards/`](prds/phase2/shards/) (01–17) |
| [`epics/phase1/epics-phase1.md`](epics/phase1/epics-phase1.md) | [`epics/phase1/shards/`](epics/phase1/shards/) (epic-01–08, phase-3 docs) |
| [`epics/phase2/epics-phase2.md`](epics/phase2/epics-phase2.md) | [`epics/phase2/shards/`](epics/phase2/shards/) (epic-09–15) |

**Rule:** Monoliths are authoritative. Shards are **read-only views** — do not edit shards directly.

---

## Agent checklist (after monolith edit)

1. **Save the monolith** — ensure post-launch sections §3.8–3.10 stay in `prd-phase1.md` when Epic 6–8 requirements change.
2. **Regenerate affected shards** — use Option A or B below.
3. **Add/update banner** on every regenerated shard:
   ```markdown
   <!-- generated from prds/phase1/prd-phase1.md YYYY-MM-DD — do not edit -->
   ```
4. **Update shard README** if files were added or renamed ([`prds/phase1/shards/README.md`](prds/phase1/shards/README.md), [`prds/phase2/shards/README.md`](prds/phase2/shards/README.md), [`epics/README.md`](epics/README.md)).
5. **Update [`INDEX.md`](INDEX.md)** task table if lookup paths changed.
6. **Grep for broken links** — `rg 'planning-artifacts/prd\.md|epics\.md|ux-japan_winter_sport'` from repo root.

---

## Option A — Curated shards (current layout)

Shards `01`–`18` map to specific PRD sections (see [`prds/phase1/shards/README.md`](prds/phase1/shards/README.md)). After editing the monolith:

1. Open the monolith section that maps to each shard (e.g. §3.8 → `16-phase1-improvements.md`).
2. Copy the section content into the shard file; keep shard frontmatter and cross-links.
3. Fix relative links for shard location (`../../epics/phase1/shards/`, `../../../docs/`, etc.).

**Epic shards:** Same process — extract epic blocks from `epics-phase1.md` / `epics-phase2.md` into `epics/phaseN/shards/epic-NN-*.md`.

---

## Option B — Full explode (new documents only)

For a **new** large markdown doc, use the BMAD shard skill:

```bash
cd web && npx @kayvan/markdown-tree-parser explode \
  ../_powri/planning-artifacts/prds/phase1/prd-phase1.md \
  ../_powri/planning-artifacts/prds/phase1/shards/_exploded
```

Then **rename/move** exploded files to match the curated `01`–`18` naming in [`prds/phase1/shards/README.md`](prds/phase1/shards/README.md). Do not replace the curated index blindly — the explode tool splits on every `##` heading.

---

## Phase 1 PRD shard map

| Shard | Monolith section |
|-------|------------------|
| `01-overview-and-goals.md` | §1–2 |
| `02-phase1-functional-requirements.md` | §3 FR list |
| `03`–`07` | §3.1–3.7 features |
| `08-ai-content.md` | §4 |
| `09-security-and-architecture.md` | §6–7 |
| `10-content-and-decisions.md` | §8, §11 |
| `11-roadmap-scope.md` | §9–10 |
| `12`–`14` | §12 UX |
| `15-analytics-legal-glossary.md` | §13, §17–18 |
| `16-phase1-improvements.md` | **§3.8** Epic 6 |
| `17-phase1-improvements-wave2.md` | **§3.9** Epic 7 |
| `18-quiz-cosmic-story-ux.md` | **§3.10** Epic 8 |

---

## Phase 2 PRD shard map

| Shard | Monolith section |
|-------|------------------|
| `01-overview-vision-users.md` | §0–2 |
| `02-glossary-and-priority.md` | §3–4 |
| `03-phase2-functional-requirements.md` | §5 FR list (FR-8–17) |
| `04-features-auth.md` | §5.1 FR-8 |
| `05-features-resort-map.md` | §5.2 FR-9 |
| `06-features-reviews-comments.md` | §5.3 FR-10 |
| `07-features-ski-passport.md` | §5.4 FR-11 |
| `08-features-trip-skeleton.md` | §5.5 FR-12 |
| `09-features-ai-stubs.md` | §5.6 FR-13 |
| `10-features-seo.md` | §5.7 FR-14 |
| `11-features-saved-resorts.md` | §5.8 FR-15 |
| `12-features-navigation.md` | §5.9 FR-16 |
| `13-features-public-profile.md` | §5.10 FR-17 |
| `14-nfrs-and-architecture.md` | §6–7 |
| `15-scope-and-non-goals.md` | §8–9 |
| `16-metrics-analytics-decisions.md` | §10–12 |
| `17-release-gates-and-readiness.md` | §13–14 |

---

## Epic numbering

Epic numbers **continue across phases** (Phase 1: 1–8; Phase 2: 9–15; Phase 3 deferred: Epic 4 AI + phase-3 shard docs).
