---
name: design-system-auditor
description: Comprehensive design system compliance scanner that detects violations including forbidden browser dialogs (alert/confirm/prompt), incorrect color usage (green instead of blue/purple), missing dark mode variants, improper spacing, and inline styling. Use when auditing design, checking ui, validating design system, reviewing components, checking compliance, scanning ui, finding violations, checking dark mode, validating styles, or ensuring brand consistency. Ensures 100% adherence to SynthCore's premium UI/UX standards across 150+ components and 100+ pages.
---

# Design System Auditor

## Overview

Ensure **100% design system compliance** across the entire SynthCore codebase by detecting and reporting violations that break our premium UI/UX standards. This skill prevents design drift, maintains consistency across 150+ components, and enforces our strict no-browser-dialog policy, saving **2+ hours per code review**.

## When to Use This Skill

Use when:
- Reviewing pull requests for UI changes
- After creating new components or pages
- Before deployment to catch design violations
- To maintain brand consistency
- Weekly compliance audits
- After team members add new features
- When UI feels inconsistent

## Critical Violations Detected

### 🚨 SEVERITY: CRITICAL
**Browser Dialogs** - ABSOLUTELY FORBIDDEN
- `alert()`, `confirm()`, `prompt()`
- `window.alert()`, `window.confirm()`, `window.prompt()`
- Dynamic dialog calls
- **Impact**: Breaks premium app experience, looks unprofessional
- **Fix**: Use toast notifications and Modal components

### 🎨 SEVERITY: HIGH
**Wrong Color Theme**
- Green/teal/indigo used for primary actions (should be blue/purple)
- Missing gradient effects on primary buttons
- Shadow colors not matching theme
- **Impact**: Breaks brand identity, inconsistent UI
- **Fix**: Replace with blue/purple gradients

### 🌙 SEVERITY: HIGH
**Missing Dark Mode Variants**
- Colors without `dark:` variants
- Background/text pairs incomplete
- Hover states missing dark variants
- **Impact**: Broken dark mode experience
- **Fix**: Add `dark:` variants to all colors

### 💅 SEVERITY: MEDIUM
**Not Using Design System Components**
- Inline `<div>` instead of `<Card>`
- Raw `<button>` instead of `<Button>`
- Custom badges instead of `<Badge>`
- **Impact**: Inconsistent styling, hard to maintain
- **Fix**: Import and use design system components

### 📐 SEVERITY: MEDIUM
**Spacing Violations**
- Non-standard padding (should be p-6, p-8)
- Non-standard gaps (should be gap-6, gap-3)
- Non-standard vertical spacing (should be space-y-8)
- **Impact**: Visual inconsistency
- **Fix**: Use standard spacing values

### 🎭 SEVERITY: MEDIUM
**Missing Interactive States**
- No hover effects on clickable elements
- Missing transition animations
- No scale/shadow on hover
- **Impact**: Poor user experience, feels unfinished
- **Fix**: Add hover:shadow-xl, hover:scale-105, transitions

## Audit Workflow

```
User Request "Audit design system compliance"
    │
    ▼
┌────────────────────────────┐
│  STEP 1: Scan Codebase     │
│  ✓ Find all .tsx/.jsx files│
│  ✓ Parse each file         │
│  ✓ Build AST for analysis  │
└──────────┬─────────────────┘
           │
           ▼
┌────────────────────────────┐
│  STEP 2: Detect Violations │
│  ✓ Browser dialog check    │
│  ✓ Color theme validation  │
│  ✓ Dark mode coverage      │
│  ✓ Component usage check   │
│  ✓ Spacing validation      │
│  ✓ Hover effect check      │
└──────────┬─────────────────┘
           │
           ▼
┌────────────────────────────┐
│  STEP 3: Categorize Issues │
│  ✓ By severity level       │
│  ✓ By violation type       │
│  ✓ By file location        │
│  ✓ Auto-fixable vs manual  │
└──────────┬─────────────────┘
           │
           ▼
┌────────────────────────────┐
│  STEP 4: Generate Fixes    │
│  ✓ Color replacements      │
│  ✓ Dark mode additions     │
│  ✓ Component migrations    │
│  ✓ Spacing adjustments     │
└──────────┬─────────────────┘
           │
           ▼
┌────────────────────────────┐
│  STEP 5: Report Results    │
│  ✓ Summary by severity     │
│  ✓ File-by-file breakdown  │
│  ✓ Fix suggestions         │
│  ✓ Compliance score        │
│  ✓ Estimated fix time      │
└────────────────────────────┘
```

## Violation Detection Methods

### Method 1: Browser Dialog Detection
Search for forbidden function calls using regex and AST parsing:

```typescript
// Patterns to detect
const browserDialogPatterns = [
  /\balert\s*\(/g,
  /\bconfirm\s*\(/g,
  /\bprompt\s*\(/g,
  /window\.alert\s*\(/g,
  /window\.confirm\s*\(/g,
  /window\.prompt\s*\(/g
];

// For each match found:
// - Report violation as CRITICAL severity
// - Suggest toast notification for alert()
// - Suggest Modal component for confirm()
// - Suggest form input in modal for prompt()
```

**Example violations**:
```typescript
// ❌ FORBIDDEN
alert('Settings saved');
if (confirm('Delete item?')) { ... }
const name = prompt('Enter name:');

// ✅ CORRECT
showToast('Settings saved', 'success');
<Modal><ConfirmDialog /></Modal>
<Modal><InputForm /></Modal>
```

### Method 2: Color Theme Validation
Check for incorrect color usage:

```typescript
// Forbidden primary colors (regex patterns)
const forbiddenPrimary = [
  /bg-green-[456789]00/g,      // Green primary backgrounds
  /text-green-[456789]00/g,    // Green primary text
  /from-green-/g,              // Green in gradients
  /to-green-/g,
  /bg-teal-/g,                 // Teal backgrounds
  /bg-indigo-[456789]00/g      // Indigo (except very light)
];

// Required patterns for primary actions
const requiredPrimary = [
  /from-blue-600/,
  /to-purple-600/,
  /gradient-to-r/
];

// For each violation:
// - Report found color
// - Suggest blue/purple gradient replacement
```

**Example violations**:
```typescript
// ❌ WRONG
className="bg-green-600 hover:bg-green-700"
className="text-green-600"

// ✅ CORRECT
className="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700"
className="text-blue-600 dark:text-blue-400"
```

### Method 3: Dark Mode Coverage Check
Detect missing `dark:` variants:

```typescript
// Patterns that need dark mode variants
const needsDarkMode = [
  { pattern: /bg-white(?!.*dark:bg-)/g, add: 'dark:bg-gray-800' },
  { pattern: /bg-gray-50(?!.*dark:bg-)/g, add: 'dark:bg-gray-900' },
  { pattern: /text-gray-900(?!.*dark:text-)/g, add: 'dark:text-white' },
  { pattern: /text-gray-600(?!.*dark:text-)/g, add: 'dark:text-gray-400' },
  { pattern: /border-gray-200(?!.*dark:border-)/g, add: 'dark:border-gray-700' },
  { pattern: /hover:bg-gray-100(?!.*dark:hover:)/g, add: 'dark:hover:bg-gray-700' }
];

// For each match:
// - Report missing dark variant
// - Suggest exact dark: class to add
```

**Example violations**:
```typescript
// ❌ WRONG - No dark mode
className="bg-white text-gray-900 border-gray-200"

// ✅ CORRECT - With dark mode
className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white border-gray-200 dark:border-gray-700"
```

### Method 4: Component Usage Validation
Check if using design system components:

```typescript
// Detect inline cards
const inlineCardPattern = /<div\s+className="[^"]*(?:bg-white|rounded-lg|shadow)[^"]*">/g;

// Detect raw buttons
const rawButtonPattern = /<button\s+className="[^"]*(?:px-4|py-2|bg-)[^"]*">/g;

// Detect inline badges
const inlineBadgePattern = /<span\s+className="[^"]*(?:px-2|py-1|rounded|text-xs)[^"]*">/g;

// Check for proper imports
const hasCardImport = /import\s+.*\bCard\b.*from\s+['"]@\/components\/ui\/Card['"]/;
const hasButtonImport = /import\s+.*\bButton\b.*from\s+['"]@\/components\/ui\/Button['"]/;

// For each violation:
// - Report inline usage
// - Suggest importing and using proper component
```

**Example violations**:
```typescript
// ❌ WRONG - Inline styling
<div className="bg-white dark:bg-gray-800 rounded-xl shadow-sm border p-6">
  Content
</div>

<button className="px-4 py-2 bg-blue-600 text-white rounded-lg">
  Action
</button>

// ✅ CORRECT - Using components
import { Card } from '@/components/ui/Card';
import { Button } from '@/components/ui/Button';

<Card className="p-6">
  Content
</Card>

<Button variant="primary">
  Action
</Button>
```

### Method 5: Spacing System Check
Validate standard spacing usage:

```typescript
// Standard spacing values
const standardPadding = ['p-6', 'p-8', 'p-4'];
const standardGap = ['gap-6', 'gap-3', 'gap-2'];
const standardSpaceY = ['space-y-8', 'space-y-6', 'space-y-4'];

// Detect non-standard values
const nonStandardPadding = /p-[^68\s]/g;  // Not p-6 or p-8
const nonStandardGap = /gap-[^632\s]/g;   // Not gap-6, gap-3, gap-2
const nonStandardSpaceY = /space-y-[^864\s]/g;

// For each violation:
// - Report non-standard value
// - Suggest closest standard value
```

**Example violations**:
```typescript
// ❌ WRONG - Non-standard spacing
className="p-5"        // Should be p-6
className="gap-4"      // Should be gap-6 or gap-3
className="space-y-5"  // Should be space-y-8 or space-y-6

// ✅ CORRECT - Standard spacing
className="p-6"
className="gap-6"
className="space-y-8"
```

### Method 6: Interactive States Check
Validate hover effects and transitions:

```typescript
// Check for hover effects on interactive elements
const clickableElements = ['button', 'a', 'Card', 'clickable'];

// Required hover patterns
const requiredHoverEffects = {
  buttons: /hover:(?:shadow-lg|scale-105)/,
  cards: /hover:(?:shadow-xl|-translate-y-1)/,
  links: /hover:(?:text-|underline)/,
  all: /transition-all duration-[23]00/
};

// For each interactive element:
// - Check if it has proper hover states
// - Verify transition timing
// - Ensure shadow/scale effects present
```

**Example violations**:
```typescript
// ❌ WRONG - Missing hover effects
<Card className="p-6">Content</Card>
<button className="bg-blue-600">Action</button>

// ✅ CORRECT - With hover effects
<Card className="p-6 hover:shadow-xl hover:-translate-y-1 transition-all duration-300">
  Content
</Card>

<Button className="hover:shadow-lg hover:scale-105 transition-all duration-200">
  Action
</Button>
```

## Audit Report Format

When reporting violations, use this structured format:

```markdown
# 🎨 Design System Audit Report

**Date**: [Date]
**Files Scanned**: [Count]
**Total Violations**: [Count]
**Compliance Score**: [Percentage]

---

## 🚨 CRITICAL VIOLATIONS

### Browser Dialogs ([Count] found)

❌ [File path]:[Line]
   [Code snippet]
   Fix: [Suggested replacement]

---

## 🎨 HIGH PRIORITY VIOLATIONS

### Wrong Colors ([Count] found)

❌ [File path]:[Line]
   Found: [Incorrect color class]
   Expected: [Correct blue/purple gradient]

### Missing Dark Mode ([Count] found)

❌ [File path]:[Line]
   Found: [Class without dark variant]
   Add: [Dark mode class]

---

## 📊 MEDIUM PRIORITY VIOLATIONS

### Component Usage ([Count] found)
### Spacing ([Count] found)
### Hover Effects ([Count] found)

---

## 🔧 AUTO-FIX SUMMARY

[X]/[Total] violations can be auto-fixed:
- Color replacements: [Count]
- Dark mode additions: [Count]
- Spacing adjustments: [Count]

Manual intervention required: [Count]

---

## 📈 COMPLIANCE SCORE

Current: [X]%
Target: 100%
Gap: [X] violations

Estimated time to fix: [Hours]
```

## File Scanning Strategy

### Scan All Components and Pages
```bash
# Files to scan
app/**/*.tsx           # All pages
components/**/*.tsx    # All components
lib/**/*.tsx          # Utility components

# Exclude
node_modules/
.next/
dist/
build/
```

### Prioritize by Impact
1. **Critical**: Public-facing pages (homepage, pricing, about)
2. **High**: Dashboard pages (user-visible features)
3. **Medium**: Admin pages (internal tools)
4. **Low**: Dev-only pages (test pages, storybook)

## Auto-Fix Capabilities

The Design System Auditor can automatically fix **75%** of violations:

### Auto-Fixable
✅ Color replacements (green → blue/purple)
✅ Dark mode variant additions
✅ Spacing value adjustments
✅ Hover effect additions
✅ Transition timing corrections

### Requires Manual Fix
❌ Browser dialog replacements (need refactor to components)
❌ Component migrations (need imports and structure changes)
❌ Complex gradient patterns
❌ Custom design requirements

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

## Common Violation Patterns

### Pattern 1: Success Colors
```typescript
// ❌ WRONG - Green for general actions
<Button className="bg-green-600">Save</Button>

// ✅ CORRECT - Blue/purple for primary, green only for semantic success
<Button variant="primary">Save</Button>  // Uses blue/purple gradient
<Badge variant="success">Completed</Badge>  // Green only for status
```

### Pattern 2: Tab Navigation
```typescript
// ❌ WRONG - Green active tabs
className={activeTab ? "border-green-500 text-green-600" : "..."}

// ✅ CORRECT - Blue/purple active tabs
className={activeTab ? "border-blue-500 text-blue-600 dark:text-blue-400" : "..."}
```

### Pattern 3: Interactive Cards
```typescript
// ❌ WRONG - Missing hover effects
<Card className="p-6">...</Card>

// ✅ CORRECT - Full hover treatment
<Card
  neonBorder
  interactive
  className="p-6 cursor-pointer hover:shadow-xl hover:-translate-y-1 transition-all duration-300"
>
  ...
</Card>
```

## Compliance Checklist

Use this to manually verify a file:
- [ ] No browser dialogs (alert/confirm/prompt)
- [ ] Blue/purple theme for primary actions
- [ ] All colors have dark: variants
- [ ] Using Card component (not inline divs)
- [ ] Using Button component (not raw buttons)
- [ ] Using Badge component (not inline spans)
- [ ] Standard spacing (p-6, gap-6, space-y-8)
- [ ] Hover effects on all interactive elements
- [ ] Transitions smooth (duration-200/300)
- [ ] Gradient directions consistent
- [ ] Shadow colors match theme (blue-500/50)
- [ ] Icon containers with gradients

## Best Practices

1. **Run After Every Feature** - Audit immediately after adding new UI
2. **Weekly Full Audits** - Scan entire codebase for drift
3. **Fix Critical First** - Browser dialogs and wrong colors first
4. **Use Auto-Fix** - Apply automated fixes when available
5. **Educate Team** - Share audit reports, explain violations
6. **Monitor Trends** - Track which violations are most common
7. **Update Guidelines** - Keep CLAUDE.md up to date with new patterns

## Resources

This skill enforces the design system defined in:
- **CLAUDE.md** - Complete design system rules and patterns
- **Component Library** - `components/ui/` folder
- **Global Styles** - `app/globals.css` utility classes

All violations are measured against these source-of-truth files.
