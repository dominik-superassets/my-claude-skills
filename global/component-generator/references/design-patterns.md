# Design Patterns

## Color Theme Requirements

### Primary Actions - Blue/Purple Gradient (MANDATORY)

```typescript
// CORRECT - Blue/purple gradient
className="bg-gradient-to-r from-blue-600 to-purple-600"
className="hover:from-blue-700 hover:to-purple-700"
className="hover:shadow-lg hover:shadow-blue-500/50"

// FORBIDDEN - Green/teal/indigo for primary
className="bg-green-600"  // Only for semantic success states
className="bg-teal-600"   // Not allowed for primary
className="bg-indigo-600" // Not allowed for primary
```

### Semantic Colors (Status Only)

| Color  | Usage                                |
|--------|--------------------------------------|
| Green  | Completed, verified, positive only   |
| Red    | Errors, failures, destructive        |
| Orange | Warnings, pending states             |

## Dark Mode Requirements

**EVERY color style MUST have a dark mode variant:**

```typescript
// CORRECT - Dark mode included
className="bg-white dark:bg-gray-800"
className="text-gray-900 dark:text-white"
className="border-gray-200 dark:border-gray-700"
className="bg-gray-50 dark:bg-gray-900"

// WRONG - Missing dark mode
className="bg-white"      // Missing dark variant
className="text-gray-900" // Missing dark variant
```

### Common Light/Dark Pairs

| Light Mode      | Dark Mode          |
|-----------------|--------------------|
| `bg-white`      | `dark:bg-gray-800` |
| `bg-gray-50`    | `dark:bg-gray-900` |
| `text-gray-900` | `dark:text-white`  |
| `text-gray-600` | `dark:text-gray-400` |
| `border-gray-200` | `dark:border-gray-700` |

## Spacing System

Standard spacing values to use consistently:

| Element        | Spacing          |
|----------------|------------------|
| Card padding   | `p-6`            |
| Gap between    | `gap-6`          |
| Vertical stack | `space-y-8`      |
| Icon size      | `h-5 w-5`        |
| Icon container | `p-2 rounded-lg` |

## Hover Effects

### Premium Button Hover
```typescript
className="hover:shadow-lg hover:shadow-blue-500/50 hover:scale-105 transition-all duration-300"
```

### Card Hover
```typescript
className="group-hover:shadow-lg group-hover:shadow-blue-500/30 transition-all duration-300"
```

### Icon Hover
```typescript
className="group-hover:text-purple-600 dark:group-hover:text-purple-400 transition-colors duration-300"
```

## Neon Effects

Available CSS utilities from globals.css:
- `neon-border-glow` - Animated border glow
- `premium-button` - Premium button styling
- `glow-primary` - Primary glow effect

Card components support `neonBorder` prop for automatic neon border application.

## Forbidden Patterns

Never use:
- Browser dialogs (`alert()`, `confirm()`, `prompt()`) - Use Modal component
- Inline `<div>` when Card exists
- Inline `<button>` when Button exists
- Colors without dark mode variants
- Green/teal for primary actions
