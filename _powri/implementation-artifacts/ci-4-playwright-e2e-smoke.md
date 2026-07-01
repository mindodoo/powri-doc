# Story CI-4: Playwright E2E smoke

Status: review

## Story

As a **product owner**,
I want Playwright smoke tests on critical user journeys in CI,
So that broken navigation or core flows are caught before merge.

## Acceptance Criteria

1. **Given** a pull request or push to `main`  
   **When** GitHub Actions runs after `npm run build`  
   **Then** `npm run test:e2e` passes against a production server (`npm run start`)

2. **And** smoke covers launch checklist §4 row 1: **Home → Explore → Quiz → Detail → Info**

3. **And** smoke covers direct loads of `/resorts/niseko-united`, `/privacy`, `/terms`, `/info`

4. **And** tests run at mobile viewport (375px-class device profile) with onboarding/consent pre-seeded via localStorage

5. **And** `docs/process/testing-strategy.md` documents `test:e2e` and browser install command

## Testing & Definition of Done

- [x] `@playwright/test` + `playwright.config.ts` with `webServer` (production start)
- [x] `npm run test:e2e` and `test:e2e:install` scripts
- [x] CI steps: install Chromium deps + run E2E after build
- [x] 3 smoke specs in `web/e2e/smoke.spec.ts`
- [x] Local pass: `build`, `test:e2e`, `lint`, `test:unit`, verify scripts
- [ ] CI green on PR (after merge)
- [ ] Manual QA: N/A (test infra only)

## Tasks / Subtasks

- [x] Add Playwright config and fixtures (AC: 1, 4)
- [x] Implement §4 journey + legal/detail smoke (AC: 2, 3)
- [x] Wire CI workflow (AC: 1)
- [x] Update testing-strategy.md + web/AGENTS.md (AC: 5)
- [x] Update sprint-status.yaml (AC: 1)

## Dev Notes

- Source: [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md) § CI rollout CI-4 · [`launch-checklist.md`](../../docs/qa/phase1/launch-checklist.md) §4 row 1
- Onboarding skipped via `onboarding_complete` localStorage; consent banner suppressed via `analytics_consent=denied`
- `prefers-reduced-motion: reduce` emulated so quiz transitions don't flake in CI
- `playwright.config.ts` and `e2e/` excluded from Next.js `tsconfig` and ESLint (Playwright fixture `use()` is not React)
- CI uses `npx playwright install --with-deps chromium` (Ubuntu deps for headless Chrome)

### References

- [Source: docs/process/testing-strategy.md § CI rollout plan]
- [Source: docs/qa/phase1/launch-checklist.md §4 Manual QA sign-off]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- 3 Playwright tests, ~6s local runtime (parallel)
- Journey test completes serious-mode quiz (4 answers) and opens top-match resort detail
- Production `next start` in CI matches Vercel runtime more closely than dev server

### File List

- `web/playwright.config.ts` (new)
- `web/e2e/fixtures.ts` (new)
- `web/e2e/smoke.spec.ts` (new)
- `web/package.json` (modified — Playwright deps + scripts)
- `web/package-lock.json` (modified)
- `web/tsconfig.json` (modified — exclude e2e/playwright config)
- `web/eslint.config.mjs` (modified — ignore e2e artifacts)
- `web/.gitignore` (modified — playwright output dirs)
- `.github/workflows/ci.yml` (modified)
- `docs/process/testing-strategy.md` (modified)
- `web/AGENTS.md` (modified)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/ci-4-playwright-e2e-smoke.md` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-30  
**Outcome:** Approve (pending CI run on PR)

### Summary

CI-4 adds minimal but high-value E2E coverage aligned with Phase 1 launch QA. Tests use accessible selectors (roles/labels from `en.json`) rather than brittle CSS. Pre-seeding storage avoids onboarding/consent flake without skipping the real UI paths under test.

### Review Findings

- [x] [Pass] E2E runs after build using `next start` — consistent with production
- [x] [Pass] Mobile viewport (Pixel 7 profile) matches §4 responsive QA intent
- [x] [Pass] Journey mirrors launch checklist §4 row 1 core path
- [x] [Pass] Legal + direct detail routes covered separately (fast failure isolation)
- [x] [Pass] Playwright output dirs gitignored; ESLint/tsconfig exclude test harness
- [x] [Note] Onboarding flow itself is not exercised — pre-seeded by design; optional future spec if onboarding regressions become a risk
- [x] [Note] CI adds ~1–2 min for browser install on Ubuntu — acceptable for PR gate; cache optimization possible later
- [x] [Low] Quiz uses fixed answer labels — i18n copy changes will break smoke (intentional guardrail; update spec when copy changes)
