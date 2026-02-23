# Auto-Fix Guide

The Design System Auditor can automatically fix **75%** of violations.

## Auto-Fixable Violations

### Color Replacements
Green/teal primary colors can be automatically replaced with blue/purple gradients.

```typescript
// Before
className="bg-green-600 hover:bg-green-700"

// After (auto-fixed)
className="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700"
```

### Dark Mode Additions
Missing dark: variants can be automatically appended.

```typescript
// Before
className="bg-white text-gray-900"

// After (auto-fixed)
className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white"
```

### Spacing Adjustments
Non-standard spacing values can be replaced with nearest standard value.

```typescript
// Before
className="p-5 gap-4"

// After (auto-fixed)
className="p-6 gap-6"
```

### Hover Effect Additions
Missing hover states can be automatically added.

```typescript
// Before
className="bg-blue-600"

// After (auto-fixed)
className="bg-blue-600 hover:shadow-lg transition-all duration-200"
```

### Transition Timing
Non-standard durations can be normalized.

```typescript
// Before
className="transition duration-150"

// After (auto-fixed)
className="transition-all duration-200"
```

## Manual Fix Required

### Browser Dialog Replacements
Require structural refactoring - cannot be auto-fixed.

```typescript
// Before (manual fix needed)
alert('Settings saved');

// Requires adding:
// 1. Toast provider import
// 2. useToast hook usage
// 3. Replacing alert() with toast()
```

### Component Migrations
Require import changes and JSX restructuring.

```typescript
// Before (manual fix needed)
<div className="bg-white rounded-xl shadow-sm p-6">
  Content
</div>

// Requires adding:
// 1. Card component import
// 2. Replacing <div> with <Card>
// 3. Moving styles to Card props
```

### Complex Gradient Patterns
Custom gradient configurations need manual review.

```typescript
// Needs manual review
className="bg-gradient-to-br from-blue-500 via-purple-500 to-pink-500"
```

### Custom Design Requirements
Any deviation from standard patterns for specific design needs.

## Auto-Fix Commands

When applying fixes, the auditor will:

1. **Preview changes** - Show before/after for each fix
2. **Batch by file** - Group fixes by file for efficient editing
3. **Preserve formatting** - Maintain code style and indentation
4. **Skip conflicts** - Flag cases where auto-fix might break functionality

## Fix Priority Order

1. **Critical fixes first** - Address browser dialogs (manual)
2. **Color corrections** - Fix brand inconsistencies
3. **Dark mode** - Add missing variants
4. **Hover effects** - Improve interactivity
5. **Spacing** - Normalize spacing values
6. **Component migration** - Replace inline patterns (manual)

## Best Practices

1. **Run After Every Feature** - Audit immediately after adding new UI
2. **Weekly Full Audits** - Scan entire codebase for drift
3. **Fix Critical First** - Browser dialogs and wrong colors first
4. **Use Auto-Fix** - Apply automated fixes when available
5. **Educate Team** - Share audit reports, explain violations
6. **Monitor Trends** - Track which violations are most common
7. **Update Guidelines** - Keep CLAUDE.md up to date with new patterns

## Summary Table

| Violation Type | Auto-Fixable | Estimated Time |
|----------------|--------------|----------------|
| Color replacements | Yes | 1 min/file |
| Dark mode additions | Yes | 1 min/file |
| Spacing adjustments | Yes | 30 sec/file |
| Hover effects | Yes | 1 min/file |
| Transition timing | Yes | 30 sec/file |
| Browser dialogs | No | 30 min each |
| Component migrations | No | 15 min each |
| Complex gradients | No | 5 min each |
