# Epic 3: Resort Detail & Getting There

**Stories:** 3.1–3.5 *(Phase 1 complete)* · **3.6 deferred → Phase 3** · **FRs:** FR-2, FR-3 · **PRD:** [`../prd/05-features-detail-getting-there.md`](../prd/05-features-detail-getting-there.md)

Photography-rich detail, highlights/lowlights, terrain, trail map link, Getting There. **No AI entry in Phase 1.**

---

## Story 3.1: Resort Detail Layout, Hero & Quick Verdict

Hero 55vh + shared-element transition; name, region pill, back; `QuickVerdict`; `resort_detail_viewed`; Unsplash attribution.

## Story 3.2: Season Snapshot, Highlights, Lowlights & Terrain

Section order: season → highlights/lowlights → terrain → trail map → getting there; T2 season labels; `StatRow`, `HighlightLowlightRow`.

## Story 3.3: Trail Map Link-Out

Official `trail_map_url` in new tab; `trail_map_opened`; `external_link_clicked` link_type trail_map.

## Story 3.4: Getting There — Transport Options

≥2 `TransportRow` per resort; clean outbound links; `getting_there_viewed`.

## Story 3.5: Gateway City Experiences & Practical Tips

≥3 gateway experience cards; collapsible practical tips (cash, onsen, JR Pass).

## Story 3.6: Floating "Plan with AI" Entry Point — **DEFERRED → Phase 3**

Moved with Epic 4 to [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md). Gate: monetisation/sponsorship before high-cost AI features.

---

**Full ACs:** [`../epics.md`](../epics.md) Epic 3 section  
**Content:** [`../../../content/CONTENT_MODEL.md`](../../../content/CONTENT_MODEL.md)
