<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §4: AI-Powered Content

**Source:** [`../prd-phase1.md`](../prd-phase1.md) §4 · **Related:** [`06-features-ai-chat.md`](06-features-ai-chat.md) · [`architecture/phase1/architecture-phase1.md`](../../../architecture/phase1/architecture-phase1.md)

---

## §4.1 Overview — seed-content first

1. **Answer from seed content** wherever possible (`content/resorts/<slug>.md`)
2. **Model for synthesis** only when needed (planning, comparisons)
3. **Live web search** only for T3 data (Phase 3) — shared TTL cache, not per-user

Model should never be first resort for info already in seed content.

---

## §4.2 Use cases (phase mapping)

| ID | Feature | Phase |
|----|---------|-------|
| 4.2a | Ask About a Resort chat | **1** (FR-6) |
| 4.2b | Plan My Trip wizard | 3 |
| 4.2c | Live snow/conditions | 3 |

---

## §4.3 Data freshness

- T1 evergreen / T2 seasonal / T3 live — see [`content/content-model.md`](../../../../../content/content-model.md)
- Always label season explicitly (e.g. "2025/26 season")
- Never present stale numbers as current
- UI shows season label + "last reviewed"

---

## §4.4 Cost guardrails (required)

### Hard limits

| Control | Limit | At limit |
|---------|-------|----------|
| Turns/session | 10 | Prompt fresh chat + summarise offer |
| Input/message | ~1,000 tokens | Trim/reject friendly |
| Output | 600–800 max_tokens | Hard API cap |
| Context | ~6 turns or summary | Summarise older |
| Web search/turn | 1 (2 max) | Phase 3 |
| Web search/session | 3 | Use cache |
| Daily/IP | ~30 msgs or 5 sessions | Rate-limit message |
| Global kill-switch | Budget threshold | Non-AI fallback |

### Cost levers

- Evergreen Q&A from seed content — no model call
- Shared cached snow fetch (Phase 3) — not per-user
- Pre-generate top FAQ per resort
- Prompt caching for system + resort context
- Model routing: Haiku for routing/FAQ, Sonnet for synthesis
- Retrieve relevant fields only — don't dump whole file

**Est. engaged session:** ~$0.05–0.10 with caps. **Full tables:** [`../prd-phase1.md`](../prd-phase1.md) §4.4.4

**Build-time AI tips:** [`../prd-phase1.md`](../prd-phase1.md) §14 (for maintaining site, not product runtime)
