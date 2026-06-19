# PRD §8 & §11: Content & Product Decisions

**Source:** [`../prd.md`](../prd.md) §8, §11 · **Schema authority:** [`../../../content/CONTENT_MODEL.md`](../../../content/CONTENT_MODEL.md)

---

## §8 Content requirements

- **20 resorts** at Phase 1 launch — all `content_status: published`
- Per-resort file: `content/resorts/<slug>.md` with YAML frontmatter
- Fields: hero, verdict, highlights, lowlights, terrain stats, transport, gateway, tags for filters/quiz

**Do not duplicate full schema here.** Use CONTENT_MODEL.md §8.1 for field definitions and T1/T2/T3 tiers.

---

## §11 Resolved decisions (Phase 1 relevant)

| Topic | Decision |
|-------|----------|
| **11.1 Monetisation** | No affiliate, booking widgets, or ads Phase 1 |
| **11.2 Photography** | Unsplash V1; attribution required (§17.2); abstract behind image interface |
| **11.3 Data accuracy** | Season labels; honest lowlights; last-reviewed signal |
| **11.4 Accounts / Plan tab** | Phase 2–3; no saved trips Phase 1 |
| **11.5 Languages** | EN only Phase 1; i18n scaffolded |
| **11.6 Partnerships** | No resort partnerships Phase 1 |
| **11.7 Content pipeline** | Git markdown → build-time import; no CMS Phase 1 |

**Clean outbound links (§11.1):** No affiliate tracking; `rel="noopener noreferrer"` on external links.

**Site hero images:** Pre-selected in `content/site-images.md`.
