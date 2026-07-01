<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD: AI Chat (FR-6) — **Phase 3 only (deferred from Phase 1)**

**Source:** [`../prd-phase1.md`](../prd-phase1.md) FR-6, §4.2a · **Epic:** [`epics/phase1/shards/phase-3-monetisation-ai.md`](../../../epics/phase1/shards/phase-3-monetisation-ai.md) · [`epics/phase1/shards/epic-04-ai-assistant.md`](../../../epics/phase1/shards/epic-04-ai-assistant.md)  
**Gate:** monetisation/sponsorship before implementation (high ongoing API cost).  
**Guardrails detail:** [`08-ai-content.md`](08-ai-content.md) · **Architecture:** [`architecture/phase1/architecture-phase1.md`](../../../architecture/phase1/architecture-phase1.md)

---

## FR-6 acceptance criteria

- [ ] Floating "Plan with AI →" on resort detail → full-screen chat
- [ ] Answers from seed content first (§4.1); **no live web search** Phase 1
- [ ] §4.4 cost guardrails (turn/token caps, per-IP limit, kill-switch)
- [ ] `ai_chat_opened`, `ai_chat_message_sent` per tracking plan

---

## Phase 1 use case (§4.2a)

Embedded chat on resort detail. Example questions:
- "Is Niseko good for beginners?"
- "Best time to visit Hakuba for powder?"
- "How do I get from CTS to Niseko without a car?"

**Knowledge order:** seed content → model synthesis → live search (Phase 3 only).

---

## Phase 1 scope exclusions

- §4.2b Plan My Trip wizard — Phase 3
- §4.2c Live snow/conditions search — Phase 3

---

## Key limits (§4.4.1 summary)

| Control | Limit |
|---------|-------|
| Max turns/session | 10 user messages |
| Max input/message | ~1,000 tokens |
| Max output | 600–800 tokens |
| Context cap | ~6 turns or rolling summary |
| Daily cap/IP | ~30 messages or 5 sessions |
| Kill-switch | Degrade to non-AI fallback |

**Full guardrails:** [`08-ai-content.md`](08-ai-content.md) or [`../prd-phase1.md`](../prd-phase1.md) §4.4
