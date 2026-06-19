# Story 6.10: PostHog Baseline Dashboards

Status: done

baseline_commit: c820d29

## Story

As a **product owner**,  
I want baseline dashboards in PostHog,  
So that launch metrics are observable from day one.

## Acceptance Criteria

1. **Given** PostHog project (EU) with live Phase 1 events  
   **When** dashboards are configured per `docs/tracking-plan.md` §5  
   **Then** five baseline views exist (wizard-created or documented with pin recipes)

2. **And** dashboard URLs recorded in `docs/analytics-verification.md` and `posthog-setup-report.md`

## Tasks / Subtasks

- [x] **Map wizard insights** to Epic 6.10 AC (discovery, quiz, feature adoption)
- [x] **Document** filter guardrail, external link CTR breakdown, satisfaction avg pin recipes (§7)
- [x] **Update** `posthog-setup-report.md` baseline table + verify checklist
- [x] **Update** `launch-checklist.md` go/no-go row

## Dev Agent Record

### Agent Model Used

Composer

### Dashboard registry

**Primary:** [Dashboard 751069](https://eu.posthog.com/project/202999/dashboard/751069)

| AC view | Status |
|---------|--------|
| Discovery funnel | Wizard: u9mwc1jI + uewhC5M5 |
| Filter guardrail | Pin recipe in analytics-verification §7 |
| External link CTR | uewhC5M5 + link_type breakdown recipe |
| Quiz funnel | Wizard: jdHRelvx |
| Satisfaction avg | Pin recipe in analytics-verification §7 |

### File List

- `docs/analytics-verification.md`
- `posthog-setup-report.md`
- `docs/launch-checklist.md`

## Senior Developer Review (AI)

**Review date:** 2026-06-16  
**Outcome:** Approve

### Review Findings

- [x] [Review][Pass] All five AC views mapped with URLs or create recipes
- [x] [Review][Pass] Primary dashboard link in both doc locations
- [x] [Review][Pass] Launch checklist go/no-go updated
