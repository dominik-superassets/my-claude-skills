# TypeScript Patterns

## Interface Structure

Always define props interface with required props first, then optional:

```typescript
export interface ComponentNameProps {
  // Required props first
  requiredProp: string;
  requiredNumber: number;

  // Optional props with ?
  optionalProp?: string;
  variant?: 'default' | 'highlight';
  className?: string;
}
```

## Common Interface Patterns

### Metric Card
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

### Form Input
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

### Data Table
```typescript
export interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  onSort?: (field: string) => void;
  sortField?: string;
  sortDirection?: 'asc' | 'desc';
}
```

### Action Handler
```typescript
export interface ActionProps {
  onAction: () => void;
  onActionAsync?: () => Promise<void>;
  disabled?: boolean;
  loading?: boolean;
}
```

## Type Safety Rules

### Avoid `any`
```typescript
// WRONG
const data: any = fetchData();

// CORRECT
const data: unknown = fetchData();
// or
const data: SpecificType = fetchData();
```

### Handle Optional Props
```typescript
// Use default values in destructuring
export function Component({
  variant = 'default',
  className = ''
}: ComponentProps) {
  // variant is now guaranteed to be string
}
```

### Union Types for Variants
```typescript
// Define specific allowed values
variant?: 'default' | 'highlight' | 'glass';
size?: 'sm' | 'md' | 'lg';
status?: 'pending' | 'active' | 'completed';
```

### Event Handler Types
```typescript
// Form events
onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;

// Click events
onClick: (e: React.MouseEvent<HTMLButtonElement>) => void;

// Simplified value handlers
onValueChange: (value: string) => void;
```

## Export Patterns

Always include displayName for debugging:

```typescript
export function ComponentName(props: ComponentNameProps) {
  // ...
}

ComponentName.displayName = 'ComponentName';
```

## Avoiding Common Errors

| Error | Solution |
|-------|----------|
| Unused imports | Remove them |
| Unused variables | Prefix with `_` or remove |
| Missing deps in useEffect | Add all dependencies |
| `@ts-ignore` | Fix the underlying type issue |
| Non-null assertion `!` | Add proper null checks |
