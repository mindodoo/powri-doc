# Epic 4: AI Resort Assistant — **Phase 3 (deferred)**

**Stories:** 4.1–4.4 · **FR:** FR-6 · **Gate:** monetisation/sponsorship · **See:** [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md)  
**PRD:** [`../prd/06-features-ai-chat.md`](../prd/06-features-ai-chat.md), [`../prd/08-ai-content.md`](../prd/08-ai-content.md)

Full-screen resort Q&A; seed content first; bounded API costs; graceful fallback. **Not in Phase 1 launch scope.**

---

## Story 4.1: AI Chat UI Overlay

Full-screen overlay from "Plan with AI →" (Story 3.6); `ChatBubble`; typing indicator; keyboard-aware input; `ai_chat_opened`.

## Story 4.2: Seed-Content Retrieval & AI Proxy

Serverless `/api/chat`; retrieve resort fields before Claude; prompt caching; Haiku routing / Sonnet synthesis; **no web search until Phase 3+**; sanitize input.

## Story 4.3: AI Cost Guardrails & Fallback

10 turns/session; token caps; ~6-turn context; IP daily limit; kill-switch → non-AI fallback; 10 req/min rate limit.

## Story 4.4: Pre-Generated FAQ Cache (Optional)

Serve top FAQs from cache with zero API call; fallback to 4.2.

---

**Architecture:** [`../architecture.md`](../architecture.md) (chat proxy, rate limiter)  
**Full ACs:** [`../epics.md`](../epics.md) Epic 4 section
