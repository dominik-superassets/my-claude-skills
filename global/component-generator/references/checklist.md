# Component Validation Checklist

## Before Delivering Component

### TypeScript & Linting
- [ ] Component renders without errors
- [ ] TypeScript compilation succeeds
- [ ] Zero linting errors
- [ ] No unused imports
- [ ] No unused variables (prefix intentional ones with `_`)
- [ ] No `any` types (use `unknown` or specific type)
- [ ] No `@ts-ignore` comments
- [ ] Props interface is properly defined and exported

### Design System Compliance
- [ ] Blue/purple gradients for primary elements (NOT green/teal)
- [ ] Dark mode variants on ALL color styles
- [ ] Standard spacing used (p-6, gap-6, space-y-8)
- [ ] Using Card component (not inline divs)
- [ ] Using Button component (not inline buttons)
- [ ] Icon containers have gradient backgrounds

### Visual & Interaction
- [ ] Hover effects with shadows and transitions
- [ ] Dark mode works correctly (test toggle)
- [ ] Responsive design works on all breakpoints
- [ ] Loading states handled
- [ ] Error states handled
- [ ] Empty states handled

### Accessibility
- [ ] ARIA labels present where needed
- [ ] Keyboard navigation works
- [ ] Focus states visible
- [ ] Color contrast sufficient

### Code Quality
- [ ] Component is exported correctly
- [ ] displayName is set
- [ ] Default props work as expected
- [ ] No console.log statements
- [ ] No hardcoded paths or values

## Quick Reference: Common Mistakes

| Mistake | Fix |
|---------|-----|
| `bg-white` without dark | Add `dark:bg-gray-800` |
| `text-gray-900` without dark | Add `dark:text-white` |
| `<div className="...">` for card | Use `<Card>` component |
| `<button>` for action | Use `<Button>` component |
| Green primary button | Change to blue/purple gradient |
| `alert()` or `confirm()` | Use Modal component |
| Missing `'use client'` | Add directive if using hooks/events |
| Unused import | Remove the import |

## Component Reference Files

Check these existing components for patterns:

**Core UI**: `components/ui/Card.tsx`, `Button.tsx`, `Badge.tsx`, `DataTable.tsx`

**Layout**: `components/dashboard/DashboardLayout.tsx`, `components/layout/Header.tsx`

**Complex**: `components/admin/AdminUserManagement.tsx` (tables, modals, state)

**Styles**: `app/globals.css` (neon-border-glow, premium-button, glow-primary)
