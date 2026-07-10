# Story CI-3: Coverage threshold 45%

Status: review

## Story

As a **product owner**,
I want a enforced 45% line-coverage gate on `web/src/lib/**`,
So that new business logic cannot land without baseline unit test protection.

## Acceptance Criteria

1. **Given** a pull request or push to `main`  
   **When** `npm run test:unit` runs in CI  
   **Then** Vitest reports coverage on `src/lib/**` and **fails if line coverage drops below 45%**

2. **And** coverage scope excludes test files, barrel `index.ts`, fixture modules, and JSON data maps

3. **And** additional unit tests bring baseline coverage to ≥ 45% without lowering the threshold

4. **And** existing CI steps (`lint`, `build`, verify scripts) still pass

## Testing & Definition of Done

- [x] `@vitest/coverage-v8` installed; `test:unit` runs with `--coverage`
- [x] `vitest.config.ts` coverage include/exclude + `thresholds.lines: 45`
- [x] Baseline unit tests added for pure lib modules (content, resort, quiz, api, analytics)
- [x] Local pass: `test:unit` (46 tests, ~45% lines), `lint`, `build`, all verify scripts
- [ ] CI green on PR (after merge)
- [ ] Manual QA: N/A (test infra only)

## Tasks / Subtasks

- [x] Add coverage provider and config (AC: 1, 2)
- [x] Add unit tests to meet 45% gate (AC: 3)
- [x] Update sprint-status.yaml (AC: 1)
- [x] Run full local verification (AC: 4)

## Dev Notes

- Source: [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md) § Coverage thresholds CI-3
- Scope: **`src/lib/**` line coverage only** — React UI deferred to CI-4 E2E
- `score.test.ts` remains a fixture module (excluded); persona coverage via `scorePersonas.test.ts`
- Analytics event registries (`phase1Events.ts`, `phase2Events.ts`) remain 0% in unit coverage — validated by `test:analytics` integration script
- Next milestone: **60%** in a dedicated PR (mid Phase 2)

### References

- [Source: docs/process/testing-strategy.md § Coverage thresholds]
- [Source: ci-2-vitest-unit-tests.md]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Baseline after CI-2 was ~34% lines; added 11 test files (+31 tests) targeting pure functions
- Final gate: **45.12%** lines on `src/lib/**` (656 lines in scope)
- `test:unit` now always runs with coverage (threshold enforced on every CI run)

### File List

- `web/vitest.config.ts` (modified — coverage config)
- `web/package.json` (modified — `@vitest/coverage-v8`, `test:unit --coverage`)
- `web/package-lock.json` (modified)
- `web/src/lib/analytics/externalLink.test.ts` (new)
- `web/src/lib/api/errors.test.ts` (new)
- `web/src/lib/content/parseBulletList.test.ts` (new)
- `web/src/lib/content/resortListItem.test.ts` (new)
- `web/src/lib/content/resortSearchItem.test.ts` (new)
- `web/src/lib/content/sortResortsByPopularity.test.ts` (new)
- `web/src/lib/content/unsplashCdn.test.ts` (new)
- `web/src/lib/quiz/matchReason.test.ts` (new)
- `web/src/lib/quiz/scoreHelpers.test.ts` (new)
- `web/src/lib/resort/aroundArea.test.ts` (modified — null fallback)
- `web/src/lib/resort/gatewayExperiences.test.ts` (new)
- `web/src/lib/resort/searchResorts.test.ts` (new)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/CI/ci-3-coverage-threshold-45.md` (new)
- `docs/process/testing-strategy.md` (modified — coverage review commands)

## Senior Developer Review (AI)

**Review date:** 2026-06-30  
**Outcome:** Approve (pending CI run on PR)

### Summary

CI-3 delivers the ratchet's first step: enforced 45% line coverage on `src/lib/**` with targeted tests on testable pure logic. Browser-bound analytics helpers and PostHog wiring remain uncovered by design until E2E/component tests or mocked unit tests are added later.

### Review Findings

- [x] [Pass] Threshold matches testing strategy (45% lines, `src/lib/**` only)
- [x] [Pass] Sensible coverage excludes (tests, index barrels, JSON maps, fixture module)
- [x] [Pass] Gate wired into existing `test:unit` CI step — no workflow duplication
- [x] [Pass] Tests focus on deterministic pure functions (search, filters, scoring helpers, content parsers)
- [x] [Note] Baseline is barely above threshold (45.12%) — new untested lib code will require tests immediately; raise to 60% in next dedicated PR
- [x] [Note] `phase1Events.ts` / `phase2Events.ts` at 0% unit coverage — acceptable while `test:analytics` guards registry drift
- [x] [Low] Persona scoring still runs in both `test:quiz` and `test:unit` — optional dedupe in future hygiene PR
