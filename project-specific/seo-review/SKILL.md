---
name: seo-review
description: >
  Audit and review SEO health of outdoor-butler.com. Analyzes pages, posts, Rank Math
  settings, schema markup, GEO readiness, and local SEO signals. Produces structured
  reports with scores and actionable recommendations. Use when user says "review SEO",
  "audit site", "check SEO health", "analyze pages", "SEO report", "what needs fixing",
  or wants to assess current SEO status before making changes.
arguments:
  - name: scope
    description: "What to review: all, pages, posts, location-pages, service-pages, blog, tags, homepage, or a specific page/post ID"
    required: false
metadata:
  author: DDD
  version: 1.0.0
  category: seo
  project: outdoor-butler
---

# SEO Review - Outdoor Butler

Audit SEO health and produce actionable reports. Philosophy: **identify what to enrich, not what to replace** — preserve what's working.

## Data Sources

- **Cached audit**: `R:\CodingWork\outdoor-butler\seo-audit-results.json`
- **Refresh**: run `R:\CodingWork\outdoor-butler\fetch-audit.ps1` (pulls from server)
- **Live API**: `GET /wp-json/wp/v2/pages` or `/posts` for current state
- **Current plan**: `R:\CodingWork\outdoor-butler\docs\plans\10022026-audit.md`
- **Execution log**: `R:\CodingWork\outdoor-butler\docs\plans\executions/`

## Review Workflow

1. Load data (cached or fresh)
2. Run scoring against criteria
3. Compare against existing plan for progress
4. Generate report
5. Highlight new issues not in current plan
6. Recommend next actions prioritized by impact

## Scoring Framework

For detailed scoring criteria, see [references/scoring-criteria.md](references/scoring-criteria.md).

### Page Health Score (0-100)

| Component | Weight | What's Checked |
|-----------|--------|----------------|
| On-Page SEO | 25% | Title, description, H1, URL, focus keyword |
| Content Quality | 25% | Word count, E-E-A-T signals, readability |
| Technical | 20% | Canonical, robots, Open Graph, encoding |
| Schema | 15% | Structured data present, valid, complete |
| GEO Readiness | 15% | AI citability, answer-first format, brand signals |

### Score Thresholds

| Range | Status | Action |
|-------|--------|--------|
| 80-100 | Excellent | Monitor, minor enrichment |
| 50-79 | Needs improvement | Schedule optimization |
| 0-49 | Poor | Prioritize for immediate fix |

## Report Structure

```markdown
## SEO Health Report - [Scope] - [Date]

### Summary
- Overall score: XX/100
- Pages reviewed: N
- Critical issues: N
- Improvements since last audit: [list]

### Critical (fix now)
[Issues with score impact, specific page/post IDs, recommended fix]

### High Priority (this week)
[...]

### Medium Priority (this month)
[...]

### What's Working Well
[Preserve these — don't change what ranks]

### Progress vs Plan
[Comparison to 10022026-audit.md plan items]
```

## Benchmark Pages

Use these well-performing pages as templates:
- Torrevieja location page (score 74)
- Elda location page (score 68)
- Crevillent location page (score 65)
- Pool Area Pressure Washing service page (score 70)

## Must-Check per Page

- [ ] SEO title set (not empty, under 60 chars)
- [ ] Meta description set (120-155 chars, no trailing whitespace)
- [ ] Focus keyword set and relevant
- [ ] No brand name typos ("Outdoor Buler")
- [ ] No character encoding corruption (â instead of ')
- [ ] Schema configured (LocalBusiness for locations, Article for blog)
- [ ] No duplicate titles/descriptions across pages

## Chain Handoff

This skill is the **entry point** of the SEO workflow chain:

```
seo-review (audit) → seo-fix (apply fixes) → seo-copy (write new content) → execution-log (record changes)
```

After completing a review:
- If issues are found that need fixing: invoke `seo-fix` with the specific page/post IDs and issue descriptions
- If content gaps are found: invoke `seo-copy` with the target page and gap details
- DO NOT fix issues yourself — produce the report and let the appropriate downstream skill handle the fix
- When orchestrated by skill-orchestrator, return structured issue lists so the orchestrator can route to the correct fix/copy skill
