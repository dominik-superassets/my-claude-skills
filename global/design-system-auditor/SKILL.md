---
name: design-system-auditor
description: Use when auditing UI components for design system compliance, checking dark mode support, validating color themes, or detecting forbidden browser dialogs (alert/confirm/prompt).
metadata:
  author: DDD
  version: 1.0.0
  category: validation
---

# Design System Auditor

Scans your codebase for design system violations, ensuring 100% adherence to UI/UX standards across all components and pages. Detects browser dialogs, incorrect colors, missing dark mode, and non-standard patterns.

## When to Use

- Reviewing pull requests for UI changes
- After creating new components or pages
- Before deployment to catch design violations
- Weekly compliance audits
- When UI feels inconsistent

## Critical Violations

### CRITICAL - Browser Dialogs (Forbidden)
- `alert()`, `confirm()`, `prompt()` - Replace with toast/Modal components

### HIGH - Wrong Color Theme
- Green/teal/indigo for primary actions - Use blue/purple gradients
- Missing gradient effects on primary buttons

### HIGH - Missing Dark Mode
- Colors without `dark:` variants
- Background/text pairs incomplete

### MEDIUM - Not Using Design System Components
- Inline `<div>` instead of `<Card>`
- Raw `<button>` instead of `<Button>`
- Custom badges instead of `<Badge>`

### MEDIUM - Spacing Violations
- Non-standard padding (use p-4, p-6, p-8)
- Non-standard gaps (use gap-2, gap-3, gap-6)

### MEDIUM - Missing Interactive States
- No hover effects on clickable elements
- Missing transition animations

## Files to Scan

```
app/**/*.tsx        # All pages
components/**/*.tsx # All components
lib/**/*.tsx        # Utility components
```

## Reference Documentation

See `references/` folder for detailed guidance:
- `detection-methods.md` - Regex patterns and detection methodology
- `violation-patterns.md` - Common violations with fix examples
- `audit-report-template.md` - Report format and workflow
- `auto-fix-guide.md` - Which violations can be auto-fixed
