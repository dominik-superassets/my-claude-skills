---
name: seo-fix
description: >
  Fix SEO issues on outdoor-butler.com via WordPress REST API. Handles Rank Math
  meta (titles, descriptions, focus keywords), schema markup, typos, encoding fixes,
  and GEO (Generative Engine Optimization) improvements. Use when user says "fix SEO",
  "update meta", "fix typo", "add schema", "fix description", provides a post/page ID,
  or references the SEO audit plan.
metadata:
  author: DDD
  version: 1.0.0
  category: seo
  project: outdoor-butler
---

# SEO Fix - Outdoor Butler

Fix SEO issues via the WordPress REST API. Philosophy: **enrich and improve, only replace when absolutely necessary** — this site already ranks well.

## Workflow

1. Identify the issue (from audit, user request, or review skill output)
2. Read current values: `GET /wp/v2/{posts|pages}/{id}`
3. Propose fix with before/after — get user approval for non-trivial changes
4. Apply fix: `POST /wp/v2/{posts|pages}/{id}` with updated fields
5. Verify: `GET` again to confirm
6. Log to execution file via `/execution-log`

## REST API Quick Reference

Auth and base URL configured in `R:\CodingWork\outdoor-butler\seo-fix.ps1`.

```
# Read post/page
GET /wp-json/wp/v2/posts/{id}
GET /wp-json/wp/v2/pages/{id}

# Update Rank Math meta
POST /wp-json/wp/v2/posts/{id}
{ "meta": { "rank_math_title": "...", "rank_math_description": "...", "rank_math_focus_keyword": "..." } }

# Update title (careful — affects URL if slug auto-updates)
POST /wp-json/wp/v2/posts/{id}
{ "title": "New Title" }
```

For complete meta key reference, see [references/rank-math-meta.md](references/rank-math-meta.md).
For JSON-LD schema templates, see [references/schema-templates.md](references/schema-templates.md).

## Safety Rules

1. **Always GET before POST** — read current value first
2. **Log every change** — before/after in execution log
3. **Don't touch content body** unless explicitly asked
4. **Trim whitespace** — remove trailing newlines/spaces from descriptions
5. **Fix encoding** — replace â with proper apostrophes
6. **Preserve Rank Math variables** — keep `%sep%`, `%sitename%` intact
7. **Verify after update** — GET again to confirm change stuck
8. **One concern at a time** — don't batch unrelated changes

## Meta Quality Gates

| Field | Rule |
|-------|------|
| SEO title | 50-60 chars, primary keyword near start, include location |
| Meta description | 120-155 chars, compelling CTA, no trailing whitespace |
| Focus keyword | Primary first, then secondary comma-separated |
| Title tag | Unique across site, matches page intent |

## GEO Awareness

When fixing meta, also consider AI search visibility. See [references/geo-optimization.md](references/geo-optimization.md) for full GEO strategies. Key quick wins:
- Clear, quotable statements with specific facts
- Answer-first formatting for key questions
- Statistics with sources (e.g., "+34 644 466 783", service area coverage)
- Strong heading hierarchy for AI extraction

## Data Sources

- Cached audit: `R:\CodingWork\outdoor-butler\seo-audit-results.json`
- Fresh data: run `R:\CodingWork\outdoor-butler\fetch-audit.ps1`
- Fix script: `R:\CodingWork\outdoor-butler\seo-fix.ps1`
- Plan: `R:\CodingWork\outdoor-butler\docs\plans\10022026-audit.md`
- Execution log: `R:\CodingWork\outdoor-butler\docs\plans\executions\`

## Troubleshooting

### REST API returns 401/403

**Cause**: WordPress authentication token expired or mu-plugin not forwarding Authorization header.

**Fix**: Re-authenticate via the WordPress MCP adapter. If using nginx (Netcup/Plesk), verify `fix-auth-header.php` mu-plugin is active at `wp-content/mu-plugins/fix-auth-header.php`. Note: `.htaccess` has NO effect on nginx — use mu-plugins or Plesk nginx settings.

### Meta update POST succeeds but value doesn't change

**Cause**: Rank Math or another SEO plugin overrides the meta on save, or caching serves stale data.

**Fix**: After POST, wait 2-3 seconds then GET again to verify. If the value reverted, check if Rank Math auto-generates the field. Some fields (like `rank_math_title`) are auto-populated if left empty — you may need to set them explicitly to override.

### Character encoding corruption after update

**Symptom**: Apostrophes display as `â€™`, dashes as `â€"`.

**Cause**: Double-encoding — content was UTF-8 but sent as Latin-1, or vice versa.

**Fix**: Before POST, ensure the replacement string uses proper UTF-8 characters: `'` not `'`, `—` not `--`. Always verify with a GET after update to catch encoding issues immediately.

### Bulk updates partially fail

**Symptom**: Some pages update successfully, others silently fail.

**Cause**: Rate limiting or WordPress REST API timeout on consecutive rapid requests.

**Fix**: Add a 1-second delay between consecutive POST requests. Process one page at a time. If a request fails, retry once before moving to the next page. Log each result individually.

## Chain Handoff

This skill sits in the middle of the SEO workflow chain:

```
seo-review (audit) → seo-fix (apply fixes) → seo-copy (write new content) → execution-log (record changes)
```

After applying fixes:
- **Always** invoke `execution-log` to record the before/after change — DO NOT skip logging
- If the fix involves writing new copy (not just meta updates): invoke `seo-copy` for content that meets brand voice and SEO guidelines
- If multiple fixes were applied: log each as a separate execution-log entry
- Return the list of changed page/post IDs so downstream skills know what was modified
