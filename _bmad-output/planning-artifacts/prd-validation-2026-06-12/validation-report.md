# Validation Report — Japan Ski & Snowboard Trip Planner

- **PRD:** `japan_ski_prd.md`
- **Rubric:** `.agents/skills/bmad-prd/assets/prd-validation-checklist.md`
- **Run at:** 2026-06-12
- **Grade:** Good

## Overall verdict

Strong, launch-ready *substance* — content model, analytics, legal, GTM, AI cost, and release gates are well above typical solo-project PRDs. Fix **scope contradictions** (10 vs 20 resorts, quiz/AI/nav phase alignment, CMS vs markdown) and add **FR IDs + counter-metrics** before writing epics. No broken dimensions; nothing blocks starting Phase 1 discovery UI if the top-10 list and Phase 1 nav are pinned first.

## Dimension verdicts

| Dimension | Verdict |
|-----------|---------|
| Decision-readiness | adequate |
| Substance over theater | strong |
| Strategic coherence | adequate |
| Done-ness clarity | thin |
| Scope honesty | adequate |
| Downstream usability | adequate |
| Shape fit | strong |

## Findings by severity

### Critical (1)

**Scope honesty — Navigation & screen phase mismatch** (§5.3, §12.5, §11.4, §10.1)  
Bottom nav, onboarding, quiz, and AI screens are specified as if they ship together, but roadmap splits them across Phases 1–3. A builder following §12 will over-build Phase 1.  
**Fix:** Tag every §12 screen with its phase; define Phase 1 nav explicitly (recommend: Home · Explore · Info — no Plan/My Trip until Phase 2/3).

### High (6)

1. **Decision-readiness — 10 vs 20 resorts** (§3.1, §10.1, §8) — Phase 1 = top 10; content = 20. Name the 10.  
2. **Decision-readiness — CMS vs markdown** (§7.3, §8) — Pick source-of-truth pipeline.  
3. **Strategic coherence — No counter-metrics** (§2) — Add bounce rate, filter-empty rate, etc.  
4. **Done-ness — No FR IDs / acceptance criteria** (§3) — Add FR-1… with testable bullets for Phase 1.  
5. **Scope honesty — Quiz analytics vs quiz phase** (§13.3 vs §10.1 Phase 3) — Align instrumented events with shipped features.  
6. **Downstream usability — No glossary** — Add appendix or canonical pointer for T1/T2/T3, slugs, gateway terms.

### Medium (6)

1. Maps stack default undecided (§7.1)  
2. Design theme / typography conflicts (§5.1 vs §12.3)  
3. Satisfaction metric collection undefined (§2)  
4. Hero video vs Unsplash-only (§3.2 vs §11.2)  
5. Terrain map embed vs link-out (§3.2 vs §8.1)  
6. §9 "V2" vs §11.4 "Phase 2" label inconsistency  

### Low (3)

1. §5 partially duplicates §12  
2. Resort #16 naming (Nendai vs Joetsu Kokusai)  
3. project-context "brownfield" vs greenfield app  

## Recommended fix order (before epics)

1. **Pin Phase 1 scope** — 10 resort names, bottom nav, which §12 screens ship  
2. **Resolve CMS vs markdown** — one paragraph in §11  
3. **Add FR IDs** for Phase 1 features only  
4. **Add counter-metrics** to §2  
5. **Phase-tag §12 screens**  

## Reviewer files

- `review-rubric.md`
