---
name: component-generator
description: Generate React TypeScript components following SynthCore's strict design system with blue/purple gradient theme, dark mode variants, neon effects, and zero linting errors. Use when creating components, building UI, scaffolding interfaces, adding cards/buttons/forms/tables/modals, generating new React components, or implementing design system patterns. Ensures TypeScript prop typing, Tailwind CSS compliance, accessibility features, and full dark mode coverage.
---

# Component Generator

## Overview

Generate production-ready React components that strictly adhere to SynthCore's design system, automatically applying blue/purple gradients, dark mode variants, neon border effects, proper spacing, and TypeScript interfaces. Saves 30+ minutes per component while ensuring 100% design compliance and zero linting errors.

## When to Use This Skill

Use when:
- Creating new React components of any type
- Building UI elements (cards, buttons, forms, tables, modals)
- Scaffolding interfaces or page layouts
- Implementing design system patterns
- Adding components to the codebase

## Core Capabilities

### 1. Component Structure Generation
- TypeScript interfaces with proper prop typing
- Functional components with hooks support
- 'use client' directives when needed
- Export statements and display names
- No unused imports (auto-cleaned)

### 2. Design System Application
- Blue/purple gradient theme (NEVER green/teal/indigo for primary)
- Dark mode variants for ALL color styles
- Neon border and glow effects (`neonBorder`, `interiorGlow` props)
- Standard spacing system (`p-6`, `gap-6`, `space-y-8`)
- Icon containers with gradient backgrounds
- Professional hover effects with shadows and transitions

### 3. Component Variants Supported
- **Cards**: With neonBorder, interactive, variant (default|glass|gradient) options
- **Buttons**: primary, secondary, danger, ghost, link variants
- **Forms**: inputs, selects, checkboxes with validation states
- **Data displays**: tables, lists, grids with proper TypeScript typing
- **Navigation**: tabs, breadcrumbs, sidebars
- **Modals and overlays**: replacing browser dialogs

## Design System Rules (CRITICAL)

Reference the comprehensive design guidelines in `CLAUDE.md`:
- Location: `R:/CodingWork/Synthcoreai/synthcoreai/CLAUDE.md`
- Section: "UI/UX Design System Guidelines"
- Sub-sections: "Component-First Development", "Premium Color System", "Professional Hover Effects"

### Color Theme Requirements
**ALWAYS use blue/purple gradients for primary actions**:
```typescript
// ✅ CORRECT - Blue/purple gradient
className="bg-gradient-to-r from-blue-600 to-purple-600"
className="hover:from-blue-700 hover:to-purple-700"
className="hover:shadow-lg hover:shadow-blue-500/50"

// ❌ FORBIDDEN - Green/teal/indigo for primary
className="bg-green-600"  // Only for semantic success states
```

**Success/Status Colors** (semantic only):
- Green: Completed, verified, positive outcomes only
- Red: Errors, failures, destructive actions
- Orange: Warnings, pending states

### Dark Mode Requirements
**EVERY color style MUST have dark mode variant**:
```typescript
// ✅ CORRECT - Dark mode included
className="bg-white dark:bg-gray-800"
className="text-gray-900 dark:text-white"
className="border-gray-200 dark:border-gray-700"

// ❌ WRONG - Missing dark mode
className="bg-white"
className="text-gray-900"
```

### Component Usage Rules
**ALWAYS use existing components, NEVER inline styling**:
```typescript
// ✅ CORRECT - Use design system components
import { Card } from '@/components/ui/Card';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';

<Card neonBorder interactive>
  <Button variant="primary">Action</Button>
</Card>

// ❌ WRONG - Inline div/button
<div className="bg-white rounded-lg shadow p-6">
  <button className="px-4 py-2 bg-blue-600 rounded">Action</button>
</div>
```

## Component Reference Files

Study existing components for patterns before generating new ones:

**Core UI Components** (primary patterns):
- `components/ui/Card.tsx` - Card component with neonBorder and variants
- `components/ui/Button.tsx` - Button variants and premium effects
- `components/ui/Badge.tsx` - Status badges with gradient styles
- `components/ui/DataTable.tsx` - Complex data tables with sorting/filtering

**Layout Components** (structure patterns):
- `components/dashboard/DashboardLayout.tsx` - Navigation and sidebar patterns
- `components/layout/Header.tsx` - Top navigation patterns
- `components/layout/Footer.tsx` - Footer patterns

**Complex Components** (advanced patterns):
- `components/admin/AdminUserManagement.tsx` - Tables, modals, state management
- `components/esg/MetricCard.tsx` - Data visualization patterns

**Global Styles**:
- `app/globals.css` - CSS utilities: `neon-border-glow`, `premium-button`, `glow-primary`

## Generation Workflow

### Step 1: Analyze Requirements
Extract from user request:
- Component name and type
- Required props and their types
- UI elements needed (icons, forms, data display)
- Interactivity requirements

### Step 2: Load Design Patterns
Read reference components to extract:
- Structure patterns (how similar components are organized)
- Styling patterns (exact className combinations)
- TypeScript patterns (prop interface structures)
- State management patterns (hooks usage)

### Step 3: Generate Component Code
Create component following this structure:
```typescript
'use client';

import React from 'react';
import { Icon1, Icon2 } from 'lucide-react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';

export interface ComponentNameProps {
  // Required props first
  requiredProp: string;
  requiredNumber: number;

  // Optional props
  optionalProp?: string;
  variant?: 'default' | 'highlight';
  className?: string;
}

export function ComponentName({
  requiredProp,
  requiredNumber,
  optionalProp,
  variant = 'default',
  className = ''
}: ComponentNameProps) {
  return (
    <Card
      neonBorder={variant === 'highlight'}
      interactive
      className={className}
    >
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          <div className="p-2 bg-gradient-to-br from-blue-50 to-purple-100 dark:from-blue-900/30 dark:to-purple-900/10 rounded-lg group-hover:shadow-lg group-hover:shadow-blue-500/30 transition-all duration-300">
            <Icon1 className="h-5 w-5 text-blue-600 dark:text-blue-400 group-hover:text-purple-600 dark:group-hover:text-purple-400 transition-colors duration-300" />
          </div>
          {requiredProp}
        </CardTitle>
      </CardHeader>
      <CardContent>
        {/* Component content */}
      </CardContent>
    </Card>
  );
}

ComponentName.displayName = 'ComponentName';
```

### Step 4: Apply Design System
Ensure:
- ✅ Blue/purple gradients for primary elements
- ✅ Dark mode variants on all colors
- ✅ Standard spacing (p-6, gap-6, space-y-8)
- ✅ Hover effects with shadows and transitions
- ✅ Icon containers with gradient backgrounds
- ✅ Proper TypeScript typing
- ✅ No unused imports

### Step 5: Validate Output
Check:
- TypeScript compiles without errors
- No linting violations
- Dark mode works correctly
- All interactive elements have hover states
- Accessibility attributes present (ARIA labels)
- Component exports correctly

## Common Component Patterns

### Metric Card (with icon and trend)
```typescript
export interface MetricCardProps {
  title: string;
  value: number | string;
  unit?: string;
  trend?: {
    value: number;
    direction: 'up' | 'down' | 'stable';
  };
  icon?: React.ComponentType<{ className?: string }>;
}
```

### Form Input (with validation)
```typescript
export interface FormInputProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  required?: boolean;
  type?: 'text' | 'email' | 'password';
}
```

### Data Table (with sorting)
```typescript
export interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  onSort?: (field: string) => void;
  sortField?: string;
  sortDirection?: 'asc' | 'desc';
}
```

## Error Prevention

### Linting Errors to Avoid
- ❌ Unused imports → Remove automatically
- ❌ Unused variables → Prefix with `_` if intentional
- ❌ Console.log in production → Remove
- ❌ Missing dependency arrays in useEffect → Add proper deps
- ❌ Non-null assertions without checks → Add validation

### TypeScript Errors to Avoid
- ❌ Using `any` type → Use `unknown` or specific type
- ❌ Missing required props → Define in interface
- ❌ Incorrect prop types → Match interface definition
- ❌ @ts-ignore comments → Fix underlying issue

### Design System Violations to Avoid
- ❌ Green/teal primary colors → Use blue/purple
- ❌ Missing dark mode → Add dark: variants
- ❌ Inline divs instead of Card → Import and use Card
- ❌ Raw buttons instead of Button → Import and use Button
- ❌ Missing hover effects → Add shadow and scale
- ❌ Browser dialogs (alert/confirm) → NEVER use, create Modal

## Testing Checklist

Before delivering component:
- [ ] Component renders without errors
- [ ] TypeScript compilation succeeds
- [ ] Zero linting errors
- [ ] Dark mode variants work correctly
- [ ] Hover effects function properly
- [ ] Responsive design works on all breakpoints
- [ ] Keyboard navigation works
- [ ] ARIA labels present for accessibility
- [ ] Component is exported correctly
- [ ] Props are properly typed with interface
- [ ] Default props work as expected
- [ ] Edge cases handled (empty state, loading, error)

## Resources

This skill references:
- **Design System Guidelines**: See `CLAUDE.md` sections on UI/UX
- **Component Templates**: See `components/ui/` for reusable patterns
- **Global Styles**: See `app/globals.css` for utility classes

No scripts or assets are currently bundled with this skill - it leverages existing codebase patterns and generates code inline based on established design system rules.