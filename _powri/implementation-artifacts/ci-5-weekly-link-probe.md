# Story CI-5: Weekly link probe

Status: review

## Story

As a **product owner**,
I want outbound content URLs probed on a weekly schedule,
So that broken trail maps and source links are detected without slowing every PR.

## Acceptance Criteria

1. **Given** the scheduled workflow runs on `main`  
   **When** GitHub Actions executes  
   **Then** `npm run test:launch:links` runs in `web/` (HEAD probes + launch gate checks)

2. **And** the workflow is **not** added to PR quality gates (`.github/workflows/ci.yml` unchanged for link probe)

3. **And** the workflow can be triggered manually via `workflow_dispatch` (pre-release / content QA)

4. **And** `docs/process/testing-strategy.md` documents the schedule and manual run path

5. **And** CI epic rollout marks CI-5 complete

## Testing & Definition of Done

- [x] `.github/workflows/link-probe-weekly.yml` added (schedule + manual dispatch)
- [x] Weekly cron: Mondays 06:00 UTC
- [x] PR CI unchanged (no `test:launch:links` on every merge)
- [x] Local pass: `npm run test:launch:links` (with network)
- [ ] First scheduled run green on `main` (after merge)
- [ ] Manual QA: N/A (infra only)

## Tasks / Subtasks

- [x] Add scheduled workflow (AC: 1, 2, 3)
- [x] Update testing-strategy.md (AC: 4)
- [x] Update sprint-status.yaml — CI-4 done, CI-5 review (AC: 5)
- [x] Run local link probe verification (AC: 1)

## Dev Notes

- Source: [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md) § Link probing · CI rollout CI-5
- Script: [`web/scripts/verify-launch-gates.ts`](../../web/scripts/verify-launch-gates.ts) with `--check-links`
- No build required — reads sibling `content/` via content loader
- Failures may be transient (third-party downtime); investigate before changing content
- Optional follow-up: GitHub issue creation on failure (out of scope for CI-5)

### References

- [Source: docs/process/testing-strategy.md § Link probing]
- [Source: docs/qa/phase1/launch-checklist.md §1 — link probe noted as separate from PR gates]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Dedicated workflow keeps flaky/slow HEAD probes off PR path per Phase A decision
- `workflow_dispatch` supports ad-hoc runs before major content releases

### File List

- `.github/workflows/link-probe-weekly.yml` (new)
- `docs/process/testing-strategy.md` (modified)
- `_powri/implementation-artifacts/sprint-status.yaml` (modified)
- `_powri/implementation-artifacts/ci-5-weekly-link-probe.md` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-30  
**Outcome:** Approve (pending first scheduled run on main)

### Summary

CI-5 closes the Phase B CI epic with a minimal scheduled job: same verify script as local, no new dependencies, no PR CI bloat. Manual dispatch covers pre-release without waiting for Monday cron.

### Review Findings

- [x] [Pass] Separate workflow — PR `ci.yml` does not run link probe
- [x] [Pass] Full repo checkout — content loader reads `../content`
- [x] [Pass] 30-minute timeout appropriate for HEAD fan-out
- [x] [Pass] `cancel-in-progress: false` avoids aborting long probe mid-run on re-trigger
- [x] [Note] Scheduled workflows on inactive repos may pause — first run after merge may need manual `workflow_dispatch` to verify
- [x] [Note] Failures do not block merges; team should treat red weekly job as content/maintenance ticket, not release blocker
- [x] [Low] No Slack/email notification on failure — acceptable for CI-5; add alerting in a follow-up if needed
