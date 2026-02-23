---
name: execution-log
description: >
  Log completed SEO work to the execution tracking file for outdoor-butler.com.
  Records before/after values, verification status, and maps to the plan. Use when
  user says "log this", "update execution log", "record what we did", after completing
  SEO fixes, or when invoking /execution-log with a plan reference.
arguments:
  - name: plan
    description: "Plan filename (e.g., '10022026-audit'). Defaults to latest plan."
    required: false
metadata:
  author: DDD
  version: 1.0.0
  category: seo
  project: outdoor-butler
---

# Execution Log - Outdoor Butler

Track all SEO changes with before/after audit trail.

## File Locations

- Plans: `R:\CodingWork\outdoor-butler\docs\plans\`
- Logs: `R:\CodingWork\outdoor-butler\docs\plans\executions\`
- Convention: `{plan-filename}-execution.md`

## Entry Format

```markdown
### {Item#}. {Title}
- **Status**: DONE | PARTIAL | SKIPPED | BLOCKED
- **Date**: YYYY-MM-DD
- **Action**: What was done (API call, setting change, etc.)
- **Before**: Original value(s)
- **After**: New value(s)
- **Verified**: Confirmation method (API GET, browser check, etc.)
- **Notes**: Optional context
```

## Status Values

| Status | Meaning |
|--------|---------|
| DONE | Fully completed and verified |
| PARTIAL | Some sub-tasks done, others remaining |
| SKIPPED | Intentionally skipped with reason |
| BLOCKED | Cannot proceed — document blocker |
| PENDING | Not yet started |

## Rules

1. Read current log before updating — never overwrite existing entries
2. Include before/after for every change
3. Record verification method
4. Update phase status when all items complete
5. Flag deviations from plan
6. Keep entries concise but auditable

## Workflow

1. Read current execution log
2. Read plan to understand intended action
3. Append new entries for completed work
4. Update phase summaries if applicable
5. Note any new issues discovered during execution

## Troubleshooting

### Log file has conflicting entries

**Symptom**: Two entries for the same item with different "After" values.

**Cause**: Multiple sessions worked on the same plan item without checking the log first.

**Fix**: ALWAYS read the current execution log before appending. If a conflict is found, keep the most recent entry and mark the older one as superseded with a note.

### Plan item numbers don't match log entries

**Symptom**: Log references "Item 5.2" but the plan has restructured and that item is now "Item 6.1".

**Cause**: Plan was updated after work began, renumbering items.

**Fix**: Log entries should include the item TITLE as well as the number. When renumbering is detected, add a note: "Plan restructured — was Item 5.2, now Item 6.1" and continue with the new numbering.

### "Before" value missing

**Symptom**: Entry shows "After" but no "Before" — can't verify what changed.

**Cause**: Skipped the GET-before-POST step, or logged after the fact without checking original state.

**Fix**: Always capture the "Before" value BEFORE making changes. If logging retroactively, use the cached audit data (`seo-audit-results.json`) to reconstruct the original value. Mark as "Before (from cache)" to indicate it wasn't captured in real-time.
