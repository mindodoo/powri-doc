# powri-doc — AI-ready product documentation

Public documentation portfolio from the **Japan Ski & Snowboard Trip Planner** project.

**Live app:** [powri.vercel.app](https://powri.vercel.app/) · **Author:** [Mindodoo](https://github.com/mindodoo)

This repo contains **no application code** — only the planning and implementation artifacts used to ship Phase 1 with [BMAD](https://github.com/bmad-code-org/BMAD-METHOD) + AI agents (Cursor). It demonstrates how to write PRDs and downstream docs that AI assistants can pick up as focused, actionable tasks.

---

## The product (brief)

Mobile-first web app for planning Japan winter sports trips: browse 20 ski resorts, take a **Find My Resort** quiz (Serious/Cosmic modes), read detail pages with Getting There guidance, and trust/legal surfaces. Phase 1 shipped June 2026.

---

## Why this repo exists

Most PRDs are written for humans in meetings. These docs were written for **humans and AI agents**:

1. **Monolith → shards** — A ~1,500-line PRD was split into ~50–150 line files so agents load only what a task needs.
2. **Task lookup** — [`_bmad-output/planning-artifacts/INDEX.md`](_bmad-output/planning-artifacts/INDEX.md) maps work (“quiz”, “resort detail”, “launch”) to the right shards.
3. **Epics → stories → artifacts** — Each story has an implementation record; sprint status and retros show traceability from spec to shipped product.
4. **Validated before build** — PRD validation reports and an implementation-readiness gate precede coding.

If you hire for product work that must work with AI-assisted development, this repo shows that capability end-to-end.

---

## Where to start reading

| Audience | Start here |
|----------|------------|
| Recruiters / PM peers | This README → [`japan_ski_prd.md`](japan_ski_prd.md) §1–3 → [`INDEX.md`](_bmad-output/planning-artifacts/INDEX.md) |
| Engineers / AI agents | [`INDEX.md`](_bmad-output/planning-artifacts/INDEX.md) → epic shard for your area → matching story file in `implementation-artifacts/` |
| Process / delivery | [`sprint-status.yaml`](_bmad-output/implementation-artifacts/sprint-status.yaml) + any `epic-*-retro-*.md` |

---

## Document flow

```
japan_ski_prd.md (original monolith)
       ↓ shard
planning-artifacts/prd/ + epics/ + features/
       ↓ implement
implementation-artifacts/{story-id}.md
       ↓ track
sprint-status.yaml + epic retrospectives
```

---

## Repository layout

| Path | Purpose |
|------|---------|
| [`japan_ski_prd.md`](japan_ski_prd.md) | Original PRD before sharding (~1,481 lines) |
| [`_bmad-output/planning-artifacts/`](_bmad-output/planning-artifacts/) | Sharded PRD, epics, architecture, UX specs, validation |
| [`_bmad-output/implementation-artifacts/`](_bmad-output/implementation-artifacts/) | Story records, sprint status, epic retrospectives |

Some cross-links in planning docs point at operational files (tracking plan, case study, content model) that live in the private application repo — not included here.

---

## Related

- **Live product:** [powri.vercel.app](https://powri.vercel.app/)
- **Application code & content:** [japan-winter-sport](https://github.com/mindodoo/japan-winter-sport) (separate repo)

---

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — share and adapt with attribution.
