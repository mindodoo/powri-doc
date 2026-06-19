# PRD §9–10: Out of Scope & Release Gates

**Source:** [`../prd.md`](../prd.md) §9–10 · **Sprint status:** [`../../implementation-artifacts/sprint-status.yaml`](../../implementation-artifacts/sprint-status.yaml)

---

## §9 Out of scope (Phase 1 / V1)

- User accounts, saved trips, Plan tab
- Accommodation map & recommendations (Phase 2)
- Restaurant / gear rental / village explorer (Phase 2)
- Live snow search, lift status (Phase 3)
- Resort comparison (Phase 3)
- Hero video loops
- Headless CMS
- TH/JA locales (scaffold only)
- Monetisation / affiliate

---

## §10 Roadmap & gates

### Build sequence (dependency-ordered)

1. Foundation: Next.js, tokens, content loader, nav, analytics, security
2. Discovery: onboarding, home, filters, search, quiz
3. Detail + Getting There *(Stories 3.1–3.5)*
4. Info, legal, launch QA *(Epic 5)*
5. **Phase 3 (post-monetisation):** AI chat — Story 3.6 → Epic 4; user reviews — [`phase-3-user-community.md`](../epics/phase-3-user-community.md)

### Release-readiness gates (human go/no-go)

- All 20 resorts published
- Privacy + Terms live
- Analytics events verified in PostHog
- `npm audit` clean
- Legal attribution on all images
- Core Web Vitals acceptable on 4G

**Full gate checklist:** [`../epics/epic-05-launch.md`](../epics/epic-05-launch.md) Story 5.5 · [`../prd.md`](../prd.md) §10.2
