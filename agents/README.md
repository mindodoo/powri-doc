# Project agents

Custom AI agent playbooks for **Powri** (not BMAD vendor skills in `.agents/`).

| Agent | File | When to use |
|-------|------|-------------|
| Independent code quality audit (per story/PR) | [`code-quality-audit.md`](code-quality-audit.md) | After each story, independent of any in-flow review; writes to [`docs/quality/tech-debt-backlog.md`](../docs/quality/tech-debt-backlog.md) |
| E2E codebase audit | [`e2e-audit.md`](e2e-audit.md) | Quarterly, pre-release, or first audit of brownfield code |

**Session entry:** [`AGENTS.md`](../AGENTS.md) at repo root · **App rules:** [`web/AGENTS.md`](../web/AGENTS.md)

Mirrored to [powri-doc](https://github.com/mindodoo/powri-doc) via `./scripts/sync-powri-doc.sh`.
