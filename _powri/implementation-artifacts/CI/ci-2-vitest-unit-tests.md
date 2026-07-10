# Story CI-2: Vitest unit tests

Status: review

## Story

As a **developer**,
I want Vitest unit tests on core `web/src/lib/**` logic in CI,
So that regressions in filters, around-area labels, quiz helpers, and persona scoring are caught on every PR.

## Acceptance Criteria

1. **Given** a pull request or push to `main`  
   **When** GitHub Actions runs  
   **Then** `npm run test:unit` executes in `web/` and passes

2. **And** the former `scripts/verify-around-area.ts` node:test script is migrated to Vitest (`aroundArea.test.ts`)

3. **And** unit tests cover resort filters, cosmic story-arc analytics, and quiz persona fixtures (via existing `score.test.ts` module)

4. **And** coverage thresholds are **not** enforced yet (CI-3)

5. **And** `test:around-area` remains as a convenience alias for the around-area Vitest file

## Testing & Definition of Done

- [x] `vitest` dev dependency + `vitest.config.ts` with `@/*` paths
- [x] `npm run test:unit` script
- [x] Migrate 4 around-area cases from node:test → Vitest
- [x] Add `filters.test.ts`, `storyArc.test.ts`, `scorePersonas.test.ts`
- [x] CI workflow: replace `test:around-area` step with `test:unit`
- [x] Local pass: `test:unit`, `lint`, `build`, all existing `test:*`
- [ ] CI green on PR (after merge)
- [ ] Manual QA: N/A (test infra only)

## Tasks / Subtasks

- [x] Install Vitest; add config (AC: 1)
- [x] Migrate around-area tests (AC: 2)
- [x] Add core lib unit tests (AC: 3)
- [x] Wire CI step (AC: 1)
- [x] Remove obsolete `verify-around-area.ts` (AC: 2)
- [x] Update sprint-status.yaml (AC: 1)

## Dev Notes

- Source: [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md) § CI rollout CI-2
- `score.test.ts` stays a fixture module (imported by `verify-quiz-scoring.ts`); excluded from Vitest glob; wrapped by `scorePersonas.test.ts`
- Native `resolve.tsconfigPaths` in Vitest 4 — no `vite-tsconfig-paths` plugin
- Coverage report + 45% gate deferred to **CI-3**

### References

- [Source: docs/process/testing-strategy.md § CI rollout plan]
- [Source: ci-1-github-actions-quality-gates.md]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- 15 Vitest tests across 4 files; all green locally
- `test:around-area` now runs Vitest subset for backward-compatible local/docs references
- CI runs full `test:unit` (supersedes separate around-area CI step)

### File List

- `web/vitest.config.ts` (new)
- `web/package.json` (modified — vitest, scripts)
- `web/package-lock.json` (modified)
- `web/src/lib/resort/aroundArea.test.ts` (new)
- `web/src/lib/resort/filters.test.ts` (new)
- `web/src/lib/quiz/storyArc.test.ts` (new)
- `web/src/lib/quiz/scorePersonas.test.ts` (new)
- `web/scripts/verify-around-area.ts` (deleted)
- `.github/workflows/ci.yml` (modified)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/CI/ci-2-vitest-unit-tests.md` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-30  
**Outcome:** Approve (pending CI run on PR)

### Summary

CI-2 scope is tight: Vitest foundation + migration of existing around-area coverage + targeted unit tests on high-value lib modules. No coverage gate (CI-3), no E2E (CI-4). Persona scoring runs twice in CI today (`test:quiz` tsx script + `scorePersonas.test.ts`) — acceptable overlap for CI-2; dedupe optional in CI-3.

### Review Findings

- [x] [Pass] Vitest config excludes `score.test.ts` fixture module — avoids double-discovery / empty test file
- [x] [Pass] `@/*` alias works for `filters.test.ts` imports
- [x] [Pass] Around-area cases preserved 1:1 from node:test script
- [x] [Pass] CI step renamed to "Unit tests"; no redundant around-area job
- [x] [Pass] `test:around-area` alias retained for docs/checklists referencing Epic 7.6
- [x] [Note] Persona tests run in both `test:quiz` and `test:unit` — ~500ms extra CI; consider removing `test:quiz` CI step once team trusts Vitest wrapper (optional, not blocking)
- [x] [Note] Docs still reference `npx tsx --test scripts/verify-around-area.ts` — update in a follow-up doc pass or CI-3 (non-blocking; `test:around-area` works)
- [x] [Low] `getFilterDefinition` "unknown id" test uses `'all' as const` — documents fallback behavior but doesn't test invalid id; fine for CI-2 baseline
