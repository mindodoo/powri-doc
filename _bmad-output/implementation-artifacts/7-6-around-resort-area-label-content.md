# Story 7.6: “Around Resort” Section — Area Label & Daytrip Content

Status: done

## Story

As a **detail page reader**,  
I want the daytrip section titled for the **resort area**, not the gateway airport,  
So that “Around Zao” means Yamagata/Zao Onsen — not Sendai or Narita.

## Acceptance Criteria

1. Daytrips heading uses `around_area` or derived label — not `gateway_airport` — **Pass**
2. Fallback priority: around_area → nearest_village → nearest_town → prefecture area — **Pass** (`deriveAroundArea`)
3. i18n `aroundArea` / deprecated `gatewayCity` for block — **Pass**
4. `gateway_airport` only in Getting There gateway line — **Pass**
5. `experienceFallback` no longer references gateway — **Pass**
6. `around_area` on all 20 resorts — **Pass**
7. Daytrips are resort-area experiences — **Pass** (existing content; no airport entries)

## Tasks / Subtasks

- [x] Schema + types + loader: `around_area`
- [x] `deriveAroundArea` + `GettingThereData.aroundArea`
- [x] `GatewayExperiencesBlock`: `areaLabel` prop, optional heading
- [x] i18n: `aroundArea`, `experienceFallback`, `experienceFallbackGeneric`
- [x] `CONTENT_MODEL.md` §4.8 + `_TEMPLATE.md`
- [x] Editorial: `around_area` all 20 resorts
- [x] `scripts/verify-around-area.ts` unit tests
- [x] `npm run lint`, `test:launch`, `npm run build`

## Dev Agent Record

### File List

- `web/src/lib/resort/aroundArea.ts` (new)
- `web/src/lib/resort/detailSections.ts`
- `web/src/lib/content/schema.ts`
- `web/src/lib/content/types.ts`
- `web/src/lib/content/loadResorts.ts`
- `web/src/components/resort/GatewayExperiencesBlock.tsx`
- `web/src/components/resort/GettingThereSection.tsx`
- `web/messages/en.json`
- `web/scripts/verify-around-area.ts` (new)
- `content/CONTENT_MODEL.md`
- `content/resorts/_TEMPLATE.md`
- `content/resorts/*.md` (20 resorts)

### Spot-check headings

| Resort | Gateway (unchanged) | Around heading |
|--------|---------------------|----------------|
| Zao Onsen | From Sendai (SDJ)… | Around Zao Onsen |
| Joetsu Kokusai | Tokyo / Narita… | Around Yuzawa & Minami-Uonuma |
| Niseko | New Chitose… | Around Niseko |
| Appi Kogen | Haneda / Morioka… | Around Hachimantai |

## Senior Developer Review (AI)

**Review date:** 2026-06-17  
**Outcome:** Approve

### AC traceability

| AC | Status | Evidence |
|----|--------|----------|
| Heading uses area not airport | Pass | `GatewayExperiencesBlock` → `t('aroundArea', { area })`; no `gatewayAirport` prop |
| Fallback priority | Pass | `deriveAroundArea()` + 4 unit tests |
| i18n `aroundArea`; `gatewayCity` removed | Pass | `en.json`; grep shows no runtime `gatewayCity` usage |
| Gateway airport only in `gatewayFrom` line | Pass | `GettingThereSection` gateway line unchanged |
| `experienceFallback` no gateway | Pass | `"Local side trip near {area}"` + generic fallback |
| `around_area` all 20 resorts | Pass | Content grep 20/20 |
| Resort-area daytrips | Pass | No airport names in `nearby_daytrips`; editorial unchanged |

### Review findings

- [x] [Review][Pass] `npm run lint`, `test:launch`, `npx tsx --test scripts/verify-around-area.ts`, build green
- [x] [Review][Pass] Schema → loader → `detailSections` → UI chain is consistent
- [x] [Review][Note] `test:launch` does not assert `around_area` presence — acceptable; consider warn if empty on published resort (future)
- [x] [Review][Note] With all 20 resorts populated, `areaLabel` null path is untested in UI — fallback `experienceFallbackGeneric` is correct defensive code
- [x] [Review][Note] Long `nearby_daytrips` card titles (Joetsu) remain; AC scoped heading fix, not card copy length
