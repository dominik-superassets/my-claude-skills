---
name: seo-copy
description: >
  Write SEO-optimized copy for outdoor-butler.com — titles, meta descriptions,
  focus keywords, and page content. Follows E-E-A-T framework, GEO citability
  principles, and brand voice guidelines. Use when user says "write copy",
  "write title", "write description", "improve content", "optimize copy",
  "write meta", or needs content for a specific page/post.
arguments:
  - name: target
    description: "Page/post name, ID, or type (e.g., 'homepage', 'location-pages', '45284')"
    required: false
metadata:
  author: DDD
  version: 1.0.0
  category: seo
  project: outdoor-butler
---

# SEO Copy - Outdoor Butler

Write SEO-optimized copy that enriches existing content. **Never replace what's working** — improve, expand, and fill gaps.

## Brand Voice

- **Tone**: Professional, approachable, confident expert
- **Not**: Salesy, pushy, generic, or AI-sounding
- **Brand name**: Outdoor Butler (never misspell)
- **Phone**: +34 644 466 783
- **Service area**: Alicante province, Orihuela Costa, Costa Blanca, Spain
- **Audience**: English-speaking expats and property owners
- **Language**: British English spelling (colour, specialise, etc.)

## Copy Rules

### SEO Titles
- 50-60 characters max
- Primary keyword near the beginning
- Include location when relevant
- Use Rank Math vars: `%sep%`, `%sitename%`
- Pattern: `[Service] in [Location] %sep% %sitename%`

### Meta Descriptions
- 120-155 characters (sweet spot)
- Include primary keyword naturally
- Compelling CTA (Contact us, Get a quote, Call now)
- Include phone when space allows
- No trailing whitespace or newlines
- Think of it as search result ad copy

### Focus Keywords
- Primary keyword first, secondary comma-separated
- Match expat search intent ("pressure washing near me Alicante")
- Include location variations: Orihuela Costa, Alicante, Costa Blanca

## GEO Citability

When writing content, optimize for AI search citation. See [references/geo-citability.md](references/geo-citability.md) for detailed guide. Key principles:
- Answer-first format (direct answer in first 40-60 words)
- Self-contained blocks of 134-167 words (optimal citation length)
- Include specific statistics and facts
- Question-based headings

## E-E-A-T Signals

For content quality criteria, see [references/eeat-framework.md](references/eeat-framework.md).
- Experience: first-hand cleaning results, before/after, local knowledge
- Expertise: cleaning techniques, equipment knowledge, material care
- Authority: years in business, areas served, customer count
- Trust: contact info, physical presence, reviews

## Content Guidelines by Page Type

| Type | Word Min | Focus |
|------|----------|-------|
| Location page | 500 | Local service delivery, area-specific needs, landmarks |
| Service page | 800 | Service benefits, process, before/after, cross-links |
| Blog post | 800-1200 | Answer a question, internal links to services |
| Homepage | 500 | Value prop, services overview, trust signals, CTA |
| Business page | 600 | Industry-specific benefits, case studies |

## Workflow

1. Read current content and SEO meta for the target
2. Identify gaps (missing meta, weak copy, no keywords)
3. Draft copy following brand voice and SEO rules
4. Present to user for approval with rationale
5. Apply via REST API (use `/seo-fix` pattern)
6. Log changes via `/execution-log`

## Chain Handoff

This skill produces content that feeds into the fix/log chain:

```
seo-review (audit) → seo-fix (apply fixes) → seo-copy (write new content) → execution-log (record changes)
```

After writing copy:
- Apply the copy via `seo-fix` patterns (REST API POST) — or invoke `seo-fix` if orchestrated
- **Always** log changes via `execution-log` with before/after values
- Return the written copy and target page/post IDs so downstream skills can apply and log
