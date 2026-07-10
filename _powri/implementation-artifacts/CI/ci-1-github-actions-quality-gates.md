# Story CI-1: GitHub Actions quality gates

Status: review

## Story

As a **product owner**,
I want automated CI on every PR and push to `main`,
So that Phase 1 launch gates run without manual error before merge.

## Acceptance Criteria

1. **Given** a pull request or push to `main`  
   **When** GitHub Actions runs  
   **Then** `lint`, `build`, `test:launch`, `test:analytics`, `test:quiz`, and around-area tests execute in `web/`

2. **And** the workflow uses placeholder env vars for build (no secrets required)

3. **And** Dependabot PRs run quality gates (unlike GitGuardian — no repo secrets needed)

4. **And** `docs/process/testing-strategy.md` CI-1 row is satisfied; link probe remains out of PR CI (CI-5)

## Testing & Definition of Done

- [x] Workflow file `.github/workflows/ci.yml` added
- [x] `npm run test:around-area` script added (wraps existing verify script)
- [x] Local pass: `lint`, `build`, all `test:*` from `web/`
- [ ] CI green on PR (after merge)
- [ ] Manual QA: N/A (infra only)

## Tasks / Subtasks

- [x] Add `.github/workflows/ci.yml` (AC: 1, 2, 3)
- [x] Add `test:around-area` npm script (AC: 1)
- [x] Update `sprint-status.yaml` epic-ci tracking
- [x] Run local verification (AC: 1)

## Dev Notes

- Source: [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md) § CI rollout CI-1
- Repo layout: app in `web/`, content in sibling `content/` — checkout repo root; `working-directory: web` for npm
- Build needs minimal `NEXT_PUBLIC_*` placeholders (see `web/.env.example`)
- Do **not** include `test:launch:links` (CI-5 — flaky external HEAD probes)
- Follow [`gitguardian-scan.yml`](../../.github/workflows/gitguardian-scan.yml) trigger pattern: `main` push + PR

### References

- [Source: docs/process/testing-strategy.md § CI rollout plan]
- [Source: docs/qa/phase1/launch-checklist.md §1 Automated checks]
- [Source: docs/process/development-workflow.md verify step]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Added `ci.yml` quality job with Node 20, npm cache, concurrency cancel-in-progress
- Placeholder env for Next.js build aligned with Vercel required vars

### File List

- `.github/workflows/ci.yml` (new)
- `web/package.json` (modified — `test:around-area`)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/CI/ci-1-github-actions-quality-gates.md` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-30  
**Outcome:** Approve (pending CI run on PR)

### Summary

Minimal CI-1 scope: wire existing verify scripts into GitHub Actions. No Vitest/coverage (CI-2/3) or Playwright (CI-4). Build placeholders avoid leaking real secrets while satisfying Next config.

### Review Findings

- [x] [Pass] Triggers match GitGuardian (`main` + PR to `main`)
- [x] [Pass] Full repo checkout — content loader reads `../content`
- [x] [Pass] Link probe excluded per testing strategy
- [x] [Note] Branch protection should require `Quality gates` check after first green run (manual GitHub setting)
