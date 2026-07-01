# Feature briefs

Feature briefs are **single-session entry points** for AI agents. Each brief collects everything needed to implement one capability without reading the full PRD or epics.

## When to use

- Starting a **new agent session** on a specific story
- Onboarding a collaborator (human or AI) to one feature area
- Code review against requirements

## Structure (template)

Each brief should include:

1. **Summary** — what, why, phase, FR/story IDs
2. **User story** — from epics
3. **Acceptance criteria** — verbatim or condensed from epics
4. **UX essentials** — screen layout, components, copy (from PRD §12.5 + `design.md`)
5. **Architecture** — routes, files, state (from the phase architecture doc (`architecture/phase1/architecture-phase1.md` or `architecture/phase2/architecture-phase2.md`))
6. **Analytics** — events from `docs/analytics/tracking-plan.md`
7. **Dependencies** — prior stories, blockers
8. **Deferred / open questions**
9. **See also** — links to shards, mocks, full § references

## Available briefs

| Brief | Story | Status |
|-------|-------|--------|
| [`quiz/brief.md`](quiz/brief.md) | 2.7 Find My Resort Quiz | Ready |
| [`quiz/scoring.md`](quiz/scoring.md) | Quiz scoring rules | Ready |

## Creating a new brief

1. Copy `quiz/brief.md` as a template
2. Pull acceptance criteria from the matching epic shard in `../epics/`
3. Pull UX from `../prds/phase1/shards/13-ux-screens.md` and `../ux-designs/ux-phase1/design.md`
4. Add a row to the lookup table in [`INDEX.md`](../INDEX.md)
