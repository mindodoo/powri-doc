<!-- generated from prds/phase1/prd-phase1.md — do not edit; run prd-shard-regen.md -->

# PRD §13, §17–18: Analytics, Legal & Glossary

**Source:** [`../prd-phase1.md`](../prd-phase1.md) §13, §17–18  
**Events authority:** [`docs/analytics/tracking-plan.md`](../../../../../docs/analytics/tracking-plan.md) ← **implement against this, not PRD tables**

---

## §13 Analytics principles

1. Instrument from launch — no retro-fitting
2. **No PII** in event properties
3. Consent-gated; `consent_state` on every event
4. Anonymous/cookieless default
5. Alias anonymous ID → user ID on account creation (Phase 2+)
6. Verify events in PostHog before feature sign-off

**Tooling:** PostHog (EU) + Plausible. Feature flags scaffold Phase 1.

### Day-one events (summary)

See tracking-plan §2 for full property lists. Key events:
`resort_card_tapped`, `filter_applied`, `filter_zero_results`, `resort_detail_viewed`, `quiz_started`, `quiz_completed`, `ai_chat_opened`, `ai_chat_message_sent`, `external_link_clicked`, `satisfaction_prompt_submitted`, `search_performed`, `getting_there_viewed`

### Adding new events

Update tracking-plan **first**, then implement (§13.5 checklist in tracking-plan §4).

---

## §17 Legal (Phase 1)

### Required pages
- Privacy Policy (`/privacy`)
- Terms of Service (`/terms`)
- Before Ring 2 public sharing

### §17.2 Unsplash attribution (mandatory)

Every image: photographer name + profile link + Unsplash link with UTM:
`utm_source=japan_ski_planner&utm_medium=referral`

Use reusable `UnsplashAttribution` component. Trigger download endpoint on first use.

### §17.3 Data handling
- Minimal collection Phase 1
- GDPR/PDPA consent
- No selling user data

---

## §18 Glossary (selected)

| Term | Meaning |
|------|---------|
| T1/T2/T3 | Content volatility tiers (evergreen / seasonal / live) |
| Seed content | Build-time markdown in `content/resorts/` |
| Gateway airport | Primary airport for reaching a resort |
| Ring 1/2/3 | GTM expansion rings (§15) |

**Full glossary:** [`../prd-phase1.md`](../prd-phase1.md) §18
