<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §1–2: Overview & Goals

**Source:** [`../prd-phase1.md`](../prd-phase1.md) §1–2 · **See also:** [`INDEX.md`](../../../INDEX.md)

---

## Vision (§1.1)

Mobile-first web app helping ski/snowboard travellers plan Japan snow trips — resort choice, accommodation, villages, gateway cities.

## Problem (§1.2)

Planning requires cross-referencing resorts, lodging, transport, food, and culture across disparate sources. No single trustworthy, up-to-date product exists.

## Target users (§1.3)

- International and domestic skiers/snowboarders
- Beginner to advanced
- Solo, couples, families, groups
- Value on-mountain experience **and** cultural exploration

## Design philosophy (§1.4)

Minimal white aesthetic, high-quality photography, clean typography, generous whitespace, premium travel magazine — **not** a booking engine.

---

## Goals & metrics (§2)

| Goal | Metric |
|------|--------|
| Right resort discovery | ≥ 70% engage with resort detail pages |
| Accommodation decisions | ≥ 40% interact with accommodation map *(Phase 2)* |
| Planning confidence | Satisfaction ≥ 4.2/5 |
| Data freshness | Resort info updated ≥ once per season |

### Guardrails (do not regress)

| Guardrail | Target |
|-----------|--------|
| Home bounce rate | ≤ 50% single-page sessions without detail view |
| Filter-zero-results | ≤ 10% of filter applications |
| External-link CTR | Baseline at launch; monitor trend |

### Satisfaction collection

Micro-survey after **2nd distinct resort detail view** in session (once max). Question: *"How helpful is this for planning your trip?"* (1–5). Event: `satisfaction_prompt_submitted`. Defer metric to Phase 2 if Phase 1 traffic too low.

**Events:** [`docs/analytics/tracking-plan.md`](../../../../../docs/analytics/tracking-plan.md)
