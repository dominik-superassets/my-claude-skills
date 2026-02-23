# Audit Report Template

When reporting violations, use this structured format:

```markdown
# Design System Audit Report

**Date**: [Date]
**Files Scanned**: [Count]
**Total Violations**: [Count]
**Compliance Score**: [Percentage]

---

## CRITICAL VIOLATIONS

### Browser Dialogs ([Count] found)

[File path]:[Line]
   [Code snippet]
   Fix: [Suggested replacement]

---

## HIGH PRIORITY VIOLATIONS

### Wrong Colors ([Count] found)

[File path]:[Line]
   Found: [Incorrect color class]
   Expected: [Correct blue/purple gradient]

### Missing Dark Mode ([Count] found)

[File path]:[Line]
   Found: [Class without dark variant]
   Add: [Dark mode class]

---

## MEDIUM PRIORITY VIOLATIONS

### Component Usage ([Count] found)
### Spacing ([Count] found)
### Hover Effects ([Count] found)

---

## AUTO-FIX SUMMARY

[X]/[Total] violations can be auto-fixed:
- Color replacements: [Count]
- Dark mode additions: [Count]
- Spacing adjustments: [Count]

Manual intervention required: [Count]

---

## COMPLIANCE SCORE

Current: [X]%
Target: 100%
Gap: [X] violations

Estimated time to fix: [Hours]
```

## Audit Workflow

```
User Request "Audit design system compliance"
    |
    v
+----------------------------+
|  STEP 1: Scan Codebase     |
|  - Find all .tsx/.jsx files|
|  - Parse each file         |
|  - Build AST for analysis  |
+------------+---------------+
             |
             v
+----------------------------+
|  STEP 2: Detect Violations |
|  - Browser dialog check    |
|  - Color theme validation  |
|  - Dark mode coverage      |
|  - Component usage check   |
|  - Spacing validation      |
|  - Hover effect check      |
+------------+---------------+
             |
             v
+----------------------------+
|  STEP 3: Categorize Issues |
|  - By severity level       |
|  - By violation type       |
|  - By file location        |
|  - Auto-fixable vs manual  |
+------------+---------------+
             |
             v
+----------------------------+
|  STEP 4: Generate Fixes    |
|  - Color replacements      |
|  - Dark mode additions     |
|  - Component migrations    |
|  - Spacing adjustments     |
+------------+---------------+
             |
             v
+----------------------------+
|  STEP 5: Report Results    |
|  - Summary by severity     |
|  - File-by-file breakdown  |
|  - Fix suggestions         |
|  - Compliance score        |
|  - Estimated fix time      |
+----------------------------+
```

## Severity Levels

| Severity | Examples | Impact |
|----------|----------|--------|
| CRITICAL | Browser dialogs | Breaks premium experience |
| HIGH | Wrong colors, missing dark mode | Breaks brand identity |
| MEDIUM | Component usage, spacing, hover | Visual inconsistency |
| LOW | Minor style deviations | Polish issues |

## Scoring Formula

```
Compliance Score = (Total Elements - Violations) / Total Elements * 100

Time Estimate:
- Critical violations: 30 min each (require refactoring)
- High violations: 5 min each (mostly replacements)
- Medium violations: 2 min each (quick fixes)
```

## Reference Files

**Design System Documentation**:
- `CLAUDE.md` - Complete design system guidelines
  - Section: "UI/UX Design System Guidelines" (primary reference)
  - Section: "Dialog & Notification Patterns" (browser dialog rules)
  - Section: "Professional Hover Effects Standards" (interaction rules)
  - Section: "Spacing System" (spacing standards)

**Component Library** (study these patterns):
- `components/ui/Card.tsx` - Card component with neonBorder
- `components/ui/Button.tsx` - Button variants (primary uses blue/purple)
- `components/ui/Badge.tsx` - Status badges with gradients
- `components/ui/Modal.tsx` - Modal for confirm/prompt replacements

**Global Styles**:
- `app/globals.css` - CSS utilities: `neon-border-glow`, `premium-button`, `glow-primary`
