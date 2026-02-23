# Detection Methods

## Method 1: Browser Dialog Detection

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
// FORBIDDEN
alert('Settings saved');
if (confirm('Delete item?')) { ... }
const name = prompt('Enter name:');

// CORRECT
showToast('Settings saved', 'success');
<Modal><ConfirmDialog /></Modal>
<Modal><InputForm /></Modal>
```

## Method 2: Color Theme Validation

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
// WRONG
className="bg-green-600 hover:bg-green-700"
className="text-green-600"

// CORRECT
className="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700"
className="text-blue-600 dark:text-blue-400"
```

## Method 3: Dark Mode Coverage Check

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
// WRONG - No dark mode
className="bg-white text-gray-900 border-gray-200"

// CORRECT - With dark mode
className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white border-gray-200 dark:border-gray-700"
```

## Method 4: Component Usage Validation

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
// WRONG - Inline styling
<div className="bg-white dark:bg-gray-800 rounded-xl shadow-sm border p-6">
  Content
</div>

<button className="px-4 py-2 bg-blue-600 text-white rounded-lg">
  Action
</button>

// CORRECT - Using components
import { Card } from '@/components/ui/Card';
import { Button } from '@/components/ui/Button';

<Card className="p-6">
  Content
</Card>

<Button variant="primary">
  Action
</Button>
```

## Method 5: Spacing System Check

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
// WRONG - Non-standard spacing
className="p-5"        // Should be p-6
className="gap-4"      // Should be gap-6 or gap-3
className="space-y-5"  // Should be space-y-8 or space-y-6

// CORRECT - Standard spacing
className="p-6"
className="gap-6"
className="space-y-8"
```

## Method 6: Interactive States Check

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
// WRONG - Missing hover effects
<Card className="p-6">Content</Card>
<button className="bg-blue-600">Action</button>

// CORRECT - With hover effects
<Card className="p-6 hover:shadow-xl hover:-translate-y-1 transition-all duration-300">
  Content
</Card>

<Button className="hover:shadow-lg hover:scale-105 transition-all duration-200">
  Action
</Button>
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
