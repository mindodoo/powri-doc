# Feature Briefs

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
4. **UX essentials** — screen layout, components, copy (from PRD §12.5 + DESIGN.md)
5. **Architecture** — routes, files, state (from architecture.md)
6. **Analytics** — events from `docs/tracking-plan.md`
7. **Dependencies** — prior stories, blockers
8. **Deferred / open questions**
9. **See also** — links to shards, mocks, full § references

## Available briefs

| Brief | Story | Status |
|-------|-------|--------|
| [`quiz.md`](quiz.md) | 2.7 Find My Resort Quiz | Ready — `/quiz` stub exists; scoring doc TBD |

## Creating a new brief

1. Copy `quiz.md` as a template
2. Pull acceptance criteria from the matching epic shard in `../epics/`
3. Pull UX from `../prd/13-ux-screens.md` and DESIGN.md
4. Add a row to the lookup table in [`../INDEX.md`](../INDEX.md)
