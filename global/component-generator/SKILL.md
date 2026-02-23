---
name: component-generator
description: Use when creating React components, building UI elements (cards, buttons, forms, tables, modals), scaffolding interfaces, or implementing design system patterns. Generates TypeScript components with Tailwind CSS, dark mode, and accessibility.
metadata:
  author: DDD
  version: 1.0.0
  category: ui-generation
---

# Component Generator

Generate production-ready React components with TypeScript interfaces, Tailwind CSS styling, dark mode variants, and accessibility features. Ensures design system compliance and zero linting errors.

## When to Use

- Creating new React components of any type
- Building UI elements (cards, buttons, forms, tables, modals)
- Scaffolding interfaces or page layouts
- Implementing design system patterns

## Core Requirements

**Design System Compliance:**
- Blue/purple gradients for primary actions (never green/teal for primary)
- Dark mode variant on every color style (`bg-white dark:bg-gray-800`)
- Standard spacing: `p-6`, `gap-6`, `space-y-8`
- Use Card/Button components, never raw divs/buttons

**TypeScript Standards:**
- Export interface for all props
- Required props first, optional with `?`
- No `any` types, no unused imports
- Include `displayName` for debugging

**Accessibility:**
- ARIA labels where needed
- Keyboard navigation support
- Visible focus states

## Generation Workflow

1. **Analyze** - Extract component name, props, UI elements needed
2. **Reference** - Check `references/` for patterns and templates
3. **Generate** - Create component following template structure
4. **Validate** - Run through checklist before delivering

## Reference Files

| File | Contents |
|------|----------|
| `references/component-template.md` | Full component structure, imports, Card/Button usage |
| `references/design-patterns.md` | Colors, dark mode pairs, spacing, hover effects |
| `references/typescript-patterns.md` | Interface patterns, type safety rules |
| `references/checklist.md` | Pre-delivery validation checklist |

## Quick Pattern Reference

**Icon container:**
```typescript
<div className="p-2 bg-gradient-to-br from-blue-50 to-purple-100 dark:from-blue-900/30 dark:to-purple-900/10 rounded-lg">
  <Icon className="h-5 w-5 text-blue-600 dark:text-blue-400" />
</div>
```

**Primary button gradient:**
```typescript
className="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700"
```

Always read the reference files for complete patterns before generating components.

## Troubleshooting

### Component renders but looks wrong in dark mode

**Cause**: Using hardcoded colors instead of CSS variables or Tailwind dark: variants.

**Fix**: Replace hardcoded hex colors with Tailwind classes. Use `dark:bg-{color}` variants. Check that the component's parent has the dark mode class propagation set up.

### TypeScript errors after generation

**Cause**: Missing imports, incorrect prop types, or using APIs not available in the project's React version.

**Fix**: Run `pnpm type-check` immediately after generation. Common issues: missing `'use client'` directive for client components in Next.js App Router, importing from wrong package paths, or using `useState`/`useEffect` without React import.

### Component doesn't match design system

**Cause**: Using raw HTML elements instead of project's UI component library (Shadcn/UI, DaisyUI).

**Fix**: Check `references/component-patterns.md` for the project's component library. Use `<Button>` not `<button>`, `<Card>` not `<div className="card">`. Run design-system-auditor after generation.

### Tailwind classes not applying

**Cause**: Dynamic class construction breaks Tailwind's purge. e.g., `bg-${color}-500` won't work.

**Fix**: Use complete class strings: `bg-red-500`, `bg-blue-500`. If dynamic, use a lookup object that maps values to full class strings. Never construct Tailwind classes with string interpolation.

## Chain Handoff

After generating a component, the following validation skills should be invoked:

1. **linting-build-validator** — Run `pnpm lint` and `pnpm type-check` to catch TypeScript errors and unused imports
2. **design-system-auditor** — Verify the generated component meets design system compliance (dark mode, correct colors, proper component usage)

When orchestrated by skill-orchestrator or used in a pipeline:
- Return the list of created/modified file paths so downstream validators know what to check
- DO NOT run validation yourself — let the orchestrator chain the appropriate validation skills
- If generating multiple components, they can be validated as a batch after all generation is complete
