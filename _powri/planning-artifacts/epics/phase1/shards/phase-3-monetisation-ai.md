<!-- generated from epics/phase1/epics-phase1.md — do not edit; run prd-shard-regen.md -->

# Phase 3: Monetisation & AI (Deferred from Phase 1)

**Gate:** Start after sponsorship / monetisation foundation is in place.  
**Rationale:** AI chat and floating CTA are high ongoing cost (API spend). Phase 1 ships a complete editorial product without AI assistance.

**Stories:** 3.6 + Epic 4 (4.1–4.4) · **FR:** FR-6 · **NFR:** NFR-9

---

## Scope

| Story | Was | Description |
|-------|-----|-------------|
| **3.6** | Epic 3 | Floating "Plan with AI →" on resort detail (UX-DR17) |
| **4.1** | Epic 4 | Full-screen AI chat overlay (`ChatBubble`, typing indicator) |
| **4.2** | Epic 4 | Seed-content retrieval + `/api/chat` proxy |
| **4.3** | Epic 4 | Cost guardrails, kill-switch, fallback |
| **4.4** | Epic 4 | Pre-generated FAQ cache (optional) |

---

## Phase 1 exclusions (confirmed 2026-06-13)

- No floating AI button on resort detail
- No AI chat UI or live `/api/chat` product surface (API scaffold from Epic 1 may remain dormant)
- No `ai_chat_opened` / `ai_chat_message_sent` instrumentation required at Phase 1 launch
- Onboarding must not promise AI (copy updated in Phase 1)

---

## Implementation order (when Phase 3 starts)

1. **3.6** — Floating entry point (stub → wires to 4.1)
2. **4.1** — Chat UI overlay
3. **4.2** — Seed retrieval + proxy
4. **4.3** — Guardrails
5. **4.4** — FAQ cache (optional)

---

**PRD:** [`prds/phase1/shards/06-features-ai-chat.md`](../../../prds/phase1/shards/06-features-ai-chat.md), [`prds/phase1/shards/08-ai-content.md`](../../../prds/phase1/shards/08-ai-content.md)  
**Full ACs:** [`epics/phase1/epics-phase1.md`](../epics-phase1.md) — Stories 3.6, Epic 4  
**Sprint:** [`_powri/implementation-artifacts/sprint-status.yaml`](../../../../implementation-artifacts/sprint-status.yaml) — `deferred`
