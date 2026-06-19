# PRD §6–7: Security & Architecture (summary)

**Source:** [`../prd.md`](../prd.md) §6–7 · **Full decisions:** [`../architecture.md`](../architecture.md) ← prefer for implementation

---

## §6 Security (Phase 1 essentials)

### Frontend
- Strict CSP; SRI on third-party assets
- No inline scripts / eval
- Sanitize search + chat inputs
- HTTPS + HSTS

### API
- **API keys server-side only** (Anthropic, Unsplash, etc.)
- Rate limits: 30 req/min search, **10 req/min AI chat** per IP
- CORS restricted to own domain

### Privacy
- GDPR/PDPA consent banner
- No PII Phase 1 beyond consent-gated analytics
- `npm audit` clean; lock file committed

---

## §7 Technical stack (recommended)

| Layer | Choice |
|-------|--------|
| Frontend | Next.js, Tailwind + design tokens (§12.2) |
| Maps | Leaflet + OSM (Phase 1–2 default) |
| State | Zustand or React Context |
| API | Serverless proxy (Vercel Edge / Lambda) |
| Content | Markdown in `content/resorts/` — build-time import |
| AI | Anthropic Claude via serverless proxy |
| Cache | Edge cache resort pages; 24h TTL snow (Phase 3) |
| Auth/DB | Supabase — **Phase 3** (Plan tab) |

**Architecture doc** has folder structure, naming, and epic→path mapping — use it over this summary when coding.
