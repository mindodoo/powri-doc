# E2E Codebase Audit Agent

> **Purpose:** One-time (then quarterly) deep audit of the entire existing codebase.  
> This is separate from `agents/code-quality-audit.md`, which handles story-by-story independent audits.  
> Run this when: onboarding to an existing codebase, before a major release, quarterly as a health check, or after a security incident.

---

## When to use this vs the story review agent

| Scenario | Use |
|---|---|
| Reviewing code written in a single story or PR | `agents/code-quality-audit.md` |
| Auditing the entire existing codebase | This document |
| First audit of a live MVP | This document — run immediately |
| Quarterly health check | This document |
| Before a major release or investor demo | This document |
| After a security incident or breach | This document — prioritise security pass only |

---

## How to run a full codebase audit

A full codebase is too large to review in a single Claude prompt. The strategy is to break it into zones, audit each zone, then consolidate findings into a single prioritised backlog.

### Step 1 — Map your codebase (10 minutes)

Before running any AI review, produce a map of what you have. Run this in your project root:

```bash
# Get a file tree (excluding node_modules, build outputs, etc.)
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.go" \) \
  ! -path "*/node_modules/*" \
  ! -path "*/.next/*" \
  ! -path "*/dist/*" \
  ! -path "*/__pycache__/*" \
  | sort > codebase-map.txt

# Count lines per file to find the largest files
wc -l $(cat codebase-map.txt) | sort -rn | head -30 > largest-files.txt

# Find most recently changed files (hot files)
git log --name-only --pretty=format: --since="90 days ago" \
  | sort | uniq -c | sort -rn | head -30 > hot-files.txt

cat codebase-map.txt | wc -l   # total file count
```

Paste `codebase-map.txt` into Claude and ask:

```
Here is my codebase file tree. Group these files into logical audit zones 
(e.g. authentication, payment, API layer, data models, frontend, config/infra).
Give me 5–10 zones with the relevant file paths listed under each.
This project is [describe your product in one sentence].
```

Save the zone map — you will use it to run each audit pass.

### Step 2 — Run the security pass first (highest risk)

Security issues in a live product are urgent. Run the security audit before anything else.

Paste each zone's code into Claude with this prompt:

```
You are running a security audit on a live production codebase (MVP, first phase).
This is not a diff review — this is a full audit of existing code.

Audit the following code for security vulnerabilities. Check every item:

AUTHENTICATION & AUTHORISATION
- Are all routes/endpoints protected that should be?
- Is the auth check happening server-side (not just client-side)?
- Are JWTs or session tokens validated correctly?
- Are there any admin or internal endpoints accessible without elevated permissions?
- Is there protection against account enumeration (login error messages)?

INJECTION & INPUT
- SQL injection: are all queries parameterised? No string concatenation?
- NoSQL injection (if applicable)
- Command injection: any exec() or shell calls with user input?
- Path traversal: any file operations with user-supplied paths?
- All user input validated and sanitised before use?

DATA EXPOSURE
- Are any secrets, API keys, or credentials hardcoded anywhere?
- Are .env files in .gitignore?
- Do API responses expose more data than the client needs?
- Is PII (names, emails, phone numbers) logged anywhere?
- Are database connection strings exposed in client-side code?

TRANSPORT & HEADERS
- Is HTTPS enforced? Any HTTP fallback?
- Are security headers set? (Content-Security-Policy, X-Frame-Options, HSTS)
- Are CORS settings locked down, or is * used?

RATE LIMITING & ABUSE
- Are public endpoints rate-limited?
- Is the login endpoint protected against brute force?
- Are file upload endpoints (if any) size and type restricted?

DEPENDENCIES
- Note any imports — flag any that look outdated or known to have CVEs.
  (Run `npm audit` or `pip-audit` separately for full dependency scan.)

For each issue found, use this format:
[CRITICAL / HIGH / MEDIUM / LOW] | [file:line if known] | [short title]
- What: description of the issue
- Risk: what an attacker could do
- Fix: specific remediation
- Deploy risk: [SAFE — fix anytime] / [STAGED — test in staging first] / [BREAKING — needs migration plan]

Zone: [zone name]
Code:
[paste zone code here]
```

### Step 3 — Run the full quality and debt pass

After security, audit for performance, quality, and maintainability zone by zone:

```
You are running a technical debt audit on a live production codebase (MVP, first phase).
The goal is to identify debt that will slow development velocity or cause production issues.
This is a full file audit, not a diff review.

Audit the following code for technical debt. Check every category:

PERFORMANCE
- N+1 database query patterns (queries inside loops)
- Missing indexes on columns used in WHERE, ORDER BY, or JOIN
- Unindexed foreign keys
- Endpoints that fetch entire tables without pagination
- Synchronous blocking calls in async code paths
- Repeated expensive operations that could be cached
- Large data payloads returned when only a subset is needed

CODE QUALITY & MAINTAINABILITY
- Logic duplicated across multiple files (DRY violations)
- Functions or classes doing too many things (over 50 lines is a signal)
- Magic numbers or hardcoded strings that should be constants
- Functions with misleading or vague names
- Missing error handling on external calls (APIs, database, file system)
- Silent error swallowing (empty catch blocks)
- Dead code — functions, routes, or variables that are never called

ARCHITECTURE
- Business logic mixed into route handlers or controllers (should be in services)
- Database queries written directly in route files (should be in a data layer)
- No clear separation between layers (API, business logic, data)
- Circular dependencies between modules
- Config or environment variables scattered across files instead of centralised

TESTS
- Files with no tests at all
- Tests that only test the happy path
- Tests that test implementation details rather than behaviour

For each issue found, use this format:
[HIGH / MEDIUM / LOW] | [file:line if known] | [short title]
- What: description
- Impact: how this affects the product or team
- Fix: specific action
- Effort: [S = under 4h] [M = 1–3 days] [L = 1+ week]
- Deploy risk: [SAFE] / [STAGED] / [BREAKING]

Zone: [zone name]
Code:
[paste zone code here]
```

### Step 4 — Consolidate findings

After all zones are audited, paste all findings into Claude and run:

```
Below are technical debt and security findings from an audit of [product name], 
a live MVP. Consolidate these into a single prioritised remediation plan.

Rules for consolidation:
1. Merge duplicate issues (same root cause, different files) into one item
2. Sort by: CRITICAL first, then HIGH, then MEDIUM, then LOW
3. Within the same severity, sort by: Security > Correctness > Performance > Quality
4. Flag any items where the fix in one place will automatically fix others (systemic issues)
5. Estimate total effort in engineer-days for each severity group
6. Identify the top 5 items to fix before the next public-facing release

Output as a clean markdown table followed by the top-5 recommendation.

Findings:
[paste all zone findings here]
```

---

## E2E audit output format

Save the final consolidated output as `docs/audits/reports/YYYY-MM-DD-report.md` using this structure:

```markdown
# Codebase audit report — [Product name]
**Date:** YYYY-MM-DD  
**Audited by:** E2E Audit Agent  
**Codebase scope:** [number of files, main tech stack]  
**Zones covered:** [list]  

---

## Executive summary (for PM)

| Severity | Count | Est. effort |
|---|---|---|
| CRITICAL | 0 | 0 days |
| HIGH | 0 | 0 days |
| MEDIUM | 0 | 0 days |
| LOW | 0 | 0 days |
| **Total** | **0** | **0 days** |

**Top 5 items to fix before next release:**
1. ...
2. ...
3. ...
4. ...
5. ...

**Biggest systemic issue:** [one sentence on the most recurring pattern]

---

## CRITICAL items (fix before next deploy)

### AUDIT-001 — [Title]
| Field | Value |
|---|---|
| Severity | CRITICAL |
| Category | Security |
| Location | `src/auth/login.js:42` |
| Effort | S (2h) |
| Deploy risk | SAFE |

**What:** ...  
**Risk:** ...  
**Fix:** ...  

---

## HIGH items (fix this sprint)
[same format]

---

## MEDIUM items (schedule in next 2 sprints)
[same format]

---

## LOW items (backlog / quarterly)
[same format]

---

## Systemic patterns found

| Pattern | Occurrences | Recommended process fix |
|---|---|---|
| Missing error handling | 12 | Add to definition of done |
| Business logic in route files | 8 | Create service layer architecture guide |
```

---

## Importing audit findings into the tech debt backlog

After the audit, bulk-import findings into `docs/quality/tech-debt-backlog.md`.

Run this prompt to generate the backlog entries:

```
Convert the following audit findings into tech debt backlog entries using this exact format:

### TD-[number] — [Short title]

| Field | Value |
|---|---|
| Severity | [CRITICAL / HIGH / MEDIUM / LOW] |
| Category | [Security / Correctness / Performance / Quality / Tests] |
| Found in | Audit: [date] |
| Date added | [today] |
| Estimated effort | [hours] |
| Deploy risk | [SAFE / STAGED / BREAKING] |
| Status | Open |

**Issue:** [one sentence]  
**Risk:** [one sentence]  
**Fix:** [specific action]  

Start numbering from TD-001. CRITICAL items first, then HIGH, MEDIUM, LOW.

Findings to convert:
[paste consolidated findings]
```

---

## What changes for a live product vs new code

| Aspect | New code (story review) | Existing live code (this audit) |
|---|---|---|
| Scope | One PR / diff | Entire codebase |
| Timing | Every story | Once to establish baseline, then quarterly |
| Risk of fixing | Low — nothing is live yet | Higher — fixes can break live users |
| Priority signal | Severity alone | Severity + how often the file is changed |
| Output | PR comment | Full audit report + backlog import |
| Who acts | Developer fixes before merge | PM prioritises, developer plans carefully |
| Testing before fix | Normal unit tests | Staging environment required for STAGED/BREAKING items |

### The deploy risk field

Every finding in a live codebase audit must include a deploy risk rating:

| Rating | Meaning | Process |
|---|---|---|
| **SAFE** | Fix is isolated, no schema changes, no API changes | Fix in a normal PR, standard review |
| **STAGED** | Fix changes behaviour that users depend on, or touches shared infrastructure | Deploy to staging first, smoke test, then production |
| **BREAKING** | Requires database migration, API version change, or affects data already in production | Full migration plan required, schedule a maintenance window or use a feature flag |

Do not fix BREAKING items without a written plan. Do not fix STAGED items directly to production.

---

## Recommended audit schedule

| Trigger | Audit type |
|---|---|
| Right now (first time) | Full audit — all zones, security + debt pass |
| Before each major release | Security pass only (fast — 2–4 hours) |
| Every quarter | Full audit — compare delta to previous report |
| After any security incident | Security pass immediately, full audit within 2 weeks |
| When onboarding a new developer or AI agent | Full audit to brief them on known debt |

---

## Running the audit without a developer

As a PM, you can run this audit yourself without writing any code. You need:

1. **Read access to the codebase** — GitHub, GitLab, or ask your developer to export the relevant files.
2. **A Claude.ai session** (Pro plan for longer context, or Projects for persistent context across a large codebase).
3. **~3–6 hours** for a typical MVP codebase (100–300 files).

Work through one zone at a time. Copy the files in each zone and paste them with the prompts above. Save each output. Then run the consolidation step.

You do not need to understand the code to run this. You are the triage layer — the AI reads the code, and you decide what to prioritise and when to fix it.

---

## Checklist — first audit of a live MVP

- [ ] Run `npm audit` or `pip-audit` and note any critical/high CVEs in dependencies
- [ ] Generate `codebase-map.txt` and `hot-files.txt` using the bash commands above
- [ ] Get zone map from Claude
- [ ] Run security pass on all zones — save raw output
- [ ] Run quality/debt pass on all zones — save raw output  
- [ ] Run consolidation prompt — save as `docs/audits/reports/YYYY-MM-DD-report.md`
- [ ] Import findings into `docs/quality/tech-debt-backlog.md`
- [ ] Identify top 5 items to fix before next release
- [ ] Schedule CRITICAL and HIGH items into the next 1–2 sprints
- [ ] Set calendar reminder for next quarterly audit
