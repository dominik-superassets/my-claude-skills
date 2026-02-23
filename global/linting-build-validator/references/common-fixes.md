# Common Issues & Fixes

## Issue 1: Unused Imports

```typescript
// WRONG - Build will fail
import { useState, useEffect, useCallback } from 'react';
import { Card, CardHeader } from '@/components/ui/Card';

export function MyComponent() {
  const [data, setData] = useState(null);
  // useEffect, useCallback, CardHeader are imported but never used!
}

// CORRECT - Only import what you use
import { useState } from 'react';
import { Card } from '@/components/ui/Card';

export function MyComponent() {
  const [data, setData] = useState(null);
}
```

**Auto-fix**: Run `pnpm lint:fix` to automatically remove unused imports.

---

## Issue 2: Unused Variables

```typescript
// WRONG - Build will fail
const fetchData = async () => {
  const response = await fetch('/api/data');
  const tempData = await response.json();  // Assigned but never used!
  return processData(data);
};

// CORRECT - Option 1: Use the variable
const fetchData = async () => {
  const response = await fetch('/api/data');
  const tempData = await response.json();
  return processData(tempData);  // Now it's used
};

// CORRECT - Option 2: Prefix with underscore if intentionally unused
const fetchData = async () => {
  const response = await fetch('/api/data');
  const _tempData = await response.json();  // Underscore signals intentional
  return processData(data);
};
```

**Auto-fix**: Skill will automatically prefix with `_` for debugging variables.

---

## Issue 3: Console.log in Production

```typescript
// WRONG - Not allowed in production
const handleSubmit = (data: FormData) => {
  console.log('Submitting:', data);  // Will fail build!
  return submitForm(data);
};

// CORRECT - Remove console.log
const handleSubmit = (data: FormData) => {
  // Use proper logging in production or remove entirely
  return submitForm(data);
};

// CORRECT - Use debug logging (stripped in production)
import { logger } from '@/lib/logger';

const handleSubmit = (data: FormData) => {
  logger.debug('Submitting:', data);  // Proper logging
  return submitForm(data);
};
```

**Auto-fix**: Skill automatically removes console.log statements.

---

## Issue 4: Floating Promises

```typescript
// WRONG - Promise not awaited
useEffect(() => {
  fetchUserData();  // Floating promise - build will fail!
}, []);

// CORRECT - Option 1: Await the promise
useEffect(() => {
  const loadData = async () => {
    await fetchUserData();
  };
  loadData();
}, []);

// CORRECT - Option 2: Void operator (fire-and-forget)
useEffect(() => {
  void fetchUserData();
}, []);

// CORRECT - Option 3: Explicit error handling
useEffect(() => {
  fetchUserData().catch(console.error);
}, []);
```

**Auto-fix**: Skill adds `void` operator for fire-and-forget operations.

---

## Issue 5: TypeScript `any` Type

```typescript
// WRONG - Using 'any'
function processData(data: any) {  // Never use 'any'!
  return data.value;
}

// CORRECT - Use 'unknown' with type guard
function processData(data: unknown) {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return (data as { value: unknown }).value;
  }
  throw new Error('Invalid data structure');
}

// CORRECT - Use specific type
interface DataStructure {
  value: string;
}

function processData(data: DataStructure) {
  return data.value;
}
```

**Manual fix required**: Must replace `any` with proper types.

---

## Issue 6: Missing Type Annotations

```typescript
// WRONG - Implicit any
export function calculateTotal(items) {  // Items type inferred as 'any'
  return items.reduce((sum, item) => sum + item.price, 0);
}

// CORRECT - Explicit types
interface Item {
  price: number;
}

export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

**Manual fix required**: Add proper TypeScript interfaces.

---

## Critical Rules Reference

Zero-tolerance policy on these issues:

1. **NO unused imports** - Causes build failure
2. **NO unused variables** - Must prefix with `_` if intentional
3. **NO `any` type** - Use `unknown` or specific types
4. **NO `@ts-ignore`** - Fix the underlying type issue
5. **NO console.log in production** - Use proper logging utilities
6. **NO floating promises** - Use `await` or `void` operator
7. **NO missing dark mode variants** - All colors need dark: variants
8. **NO browser dialogs** - Never use alert/confirm/prompt
