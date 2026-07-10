# Epic CI Retrospective — Phase B Testing & Quality Gates

**Date:** 2026-07-01  
**Epic:** CI — Automated testing pyramid + GitHub Actions  
**Status:** Complete (5 stories: CI-1–CI-5; Phase A docs in PR #9)  
**Retrospective status:** done

---

## Epic Summary

The CI epic delivered the full **Phase B** testing pyramid: GitHub Actions on every PR, Vitest unit tests with a coverage ratchet, Playwright E2E smoke, and a weekly outbound link probe. Phase A (PR #9) established [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md), story DoD, and the PR template **before** wiring automation — so each implementation story had a clear acceptance target.

**Strategy doc:** [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md)  
**Workflows:** [`.github/workflows/ci.yml`](../../.github/workflows/ci.yml), [`.github/workflows/link-probe-weekly.yml`](../../.github/workflows/link-probe-weekly.yml)

### Delivered

| Story | PR | Outcome |
|-------|-----|---------|
| **Phase A** | [#9](https://github.com/mindodoo/Powri/pull/9) | Testing strategy, DoD matrix, CI rollout plan, PR template |
| **CI-1** | [#11](https://github.com/mindodoo/Powri/pull/11) | `.github/workflows/ci.yml` — lint, build, launch/analytics/quiz/around-area gates |
| **CI-2** | [#12](https://github.com/mindodoo/Powri/pull/12) | Vitest setup; `verify-around-area.ts` → Vitest; core `src/lib/**` unit tests |
| **CI-3** | [#13](https://github.com/mindodoo/Powri/pull/13) | **45%** line coverage threshold on `src/lib/**`; coverage HTML report |
| **CI-4** | [#14](https://github.com/mindodoo/Powri/pull/14) | Playwright smoke — Home → Explore → Quiz → Detail → Info + legal + direct detail |
| **CI-5** | [#15](https://github.com/mindodoo/Powri/pull/15) | Weekly `test:launch:links` — Mondays 06:00 UTC + `workflow_dispatch` |

### Metrics

- **Stories completed:** 5 / 5 (100%)
- **Unit tests:** ~15 Vitest files, ~47 tests
- **Line coverage:** **45.12%** on `src/lib/**` (656 lines in scope) — threshold met with minimal headroom
- **E2E smoke:** 3 Playwright specs (full journey, legal routes, Niseko direct detail)
- **PR CI runtime:** Single `quality` job — lint → build → verify scripts → unit+coverage → Playwright
- **Link probe:** Out of PR CI by design; scheduled on GitHub-hosted runners (no laptop required)

---

## Pre–Epic 7 / Launch Follow-Through

| Prior item | Status | Notes |
|------------|--------|-------|
| Phase 1 launch gates (`test:launch`, etc.) | ✅ Done pre-CI | Now **automated on every PR** via CI-1 |
| Manual `npm run lint && build && test:*` before merge | ✅ Replaced | CI enforces; local run still documented in AGENTS.md |
| `verify-around-area.ts` standalone script | ✅ Migrated | CI-2 → Vitest `test:around-area` |
| Epic 7 / 8 manual QA sign-offs | ⏳ Unchanged | Manual QA complements automation — not replaced |

---

## What Went Well

1. **Docs-first (Phase A) de-risked implementation.** PR #9 locked the pyramid, DoD, and rollout table before CI-1 touched workflows — stories had unambiguous acceptance criteria.
2. **One story per PR kept reviews small.** CI-1 → CI-5 merged in order; each PR built on the previous without a monolithic “big bang” CI change.
3. **Existing verify scripts became the integration layer.** `test:launch`, `test:analytics`, and `test:quiz` were already valuable; CI-1 wired them without rewriting logic.
4. **Coverage ratchet landed at the right first step.** 45% forced tests on pure helpers (filters, parsers, around-area) without blocking on browser-bound analytics wiring.
5. **E2E scope stayed tight.** Three smoke specs cover launch checklist §4 flows; onboarding/consent pre-seeded in fixtures avoids flaky banner interactions.
6. **Link probe separation was correct.** Slow/flaky external HEAD probes run weekly — not on every PR — while Niseko detail in E2E gives representative outbound-link confidence.

---

## Challenges & Growth Areas

1. **CI-1 `npm ci` lockfile mismatch.** `@swc/helpers` resolution failed on clean CI until `"overrides": { "@swc/helpers": "0.5.23" }` in `web/package.json` — lockfile drift is a first-run-on-CI tax; consider documenting override policy.
2. **45% threshold is a tight floor.** CI-3 landed at **45.12%** — any new untested `src/lib/**` code fails CI immediately; good for discipline, brittle without habit of pairing lib changes with tests.
3. **Persona scoring runs twice.** `test:quiz` and `test:unit` (`scorePersonas.test.ts`) both exercise persona fixtures — acceptable duplication today, minor CI time cost.
4. **E2E without `data-testid`.** Selectors rely on roles, text, and routes; onboarding pre-seed via `localStorage` works but is fragile if consent/onboarding copy or keys change.
5. **Playwright + Next.js toolchain boundaries.** `playwright.config.ts` and `e2e/` excluded from app `tsconfig` and ESLint — correct, but agents must know E2E lives outside the Next compile graph.
6. **Branch protection not yet enforced in repo settings.** Workflows exist on `main`; GitHub “Required status check: Quality gates” is still a manual repo-admin step.

---

## Key Insights

1. **Test pyramid layers have different merge policies.** PR-blocking (unit, integration, E2E) vs scheduled (link probe) vs manual (epic QA) — encode the policy in `testing-strategy.md`, not only in workflow YAML.
2. **Pure-function unit tests are the fastest coverage win.** Browser/PostHog paths stay covered by integration scripts and E2E until mocked unit tests are worth the cost.
3. **Story-sized CI increments beat one mega-workflow PR.** Each story validated green CI before the next added time/complexity (especially Playwright browser install).
4. **Coverage excludes are part of the contract.** Test fixtures, index barrels, and JSON maps excluded from the denominator — document when adding new lib patterns.

---

## Deferred Work Inventory (post–Epic CI)

| Item | Source | Target |
|------|--------|--------|
| Enable **Required status check** “Quality gates” on `main` | CI-1 / this retro | GitHub repo settings |
| First **scheduled link probe** green run | CI-5 | Monday 06:00 UTC or manual `workflow_dispatch` |
| Raise coverage to **60%** | testing-strategy § Coverage milestones | Dedicated story before mid–Phase 2 |
| Dedupe persona runs (`test:quiz` vs `test:unit`) | CI-3 CR note | Optional hygiene PR |
| Pytest for `scripts/*.py` | testing-strategy | Deferred |
| Component tests / mocked analytics unit tests | CI-3 CR note | Phase 2+ as needed |
| Sync public doc mirror | AGENTS.md | `./scripts/sync-powri-doc.sh` after retro merge |

---

## Readiness Assessment

| Area | Status | Notes |
|------|--------|-------|
| PR CI (CI-1–CI-4) | ✅ Live on `main` | All merged PRs green |
| Unit + coverage (CI-2–CI-3) | ✅ Live | 45% enforced on every PR |
| E2E smoke (CI-4) | ✅ Live | Chromium in CI job |
| Weekly link probe (CI-5) | ✅ Workflow live | Await first scheduled run confirmation |
| Branch protection | ⏳ Manual | Repo admin: require “Quality gates” |
| Phase 2 feature work | ✅ Unblocked | Epic 9+ can rely on CI DoD |

**Verdict:** **Epic CI complete — automation pyramid live. Remaining items are repo-admin (branch protection) and operational verification (first link probe run).**

---

## Action Items

| # | Action | Owner | When |
|---|--------|-------|------|
| 1 | Enable required status check **Quality gates** on `main` | Repo admin | Before next feature PR merge |
| 2 | Trigger or wait for first **link-probe-weekly** green run | Dev / owner | Post-merge; use `workflow_dispatch` if impatient |
| 3 | Plan **60% coverage** story before significant Phase 2 `src/lib/**` growth | Product + dev | Mid–Phase 2 per testing strategy |
| 4 | Run `./scripts/sync-powri-doc.sh` after retro lands on `main` | Owner | Post-merge |
| 5 | Optional: dedupe persona scoring between `test:quiz` and `test:unit` | Dev | Hygiene PR when CI time matters |

---

## Commitments & Next Steps

1. Epic CI retrospective recorded; sprint status `epic-ci: done`, `epic-ci-retrospective: done`
2. Phase 2 epics (9+) use story DoD + CI as merge gate — no manual-only verify
3. Raise coverage threshold in dedicated PRs only (never lower without PO approval)
4. Resume Phase 2 implementation starting with Epic 9 (Supabase auth) per sprint plan

---

## Verification (post–Epic CI)

**PR CI equivalent (local):**

```bash
cd web && npm run lint && npm run build && npm run test:launch && npm run test:analytics && npm run test:quiz && npm run test:unit && npm run test:e2e
```

**Coverage HTML:**

```bash
cd web && npm run test:unit && open coverage/index.html
```

**Link probe (weekly job, manual):**

```bash
cd web && npm run test:launch:links
```

**GitHub Actions:**

- [CI workflow](https://github.com/mindodoo/Powri/actions/workflows/ci.yml) — every PR + push to `main`
- [Link probe weekly](https://github.com/mindodoo/Powri/actions/workflows/link-probe-weekly.yml) — Mondays 06:00 UTC
