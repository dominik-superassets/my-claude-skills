# Component Template

## Standard Component Structure

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

## Component Variants

### Card Component
```typescript
<Card neonBorder interactive>
  <Button variant="primary">Action</Button>
</Card>
```

Card props: `neonBorder`, `interactive`, `variant` (default|glass|gradient)

### Button Variants
- `primary` - Blue/purple gradient, main actions
- `secondary` - Outline style
- `danger` - Red, destructive actions
- `ghost` - Minimal, no background
- `link` - Text-only, underline on hover

## Icon Container Pattern

Standard icon container with gradient background:

```typescript
<div className="p-2 bg-gradient-to-br from-blue-50 to-purple-100 dark:from-blue-900/30 dark:to-purple-900/10 rounded-lg group-hover:shadow-lg group-hover:shadow-blue-500/30 transition-all duration-300">
  <IconName className="h-5 w-5 text-blue-600 dark:text-blue-400 group-hover:text-purple-600 dark:group-hover:text-purple-400 transition-colors duration-300" />
</div>
```

## Required Imports

Always import from design system components:

```typescript
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
```

Never use raw HTML elements (`<div>`, `<button>`) when design system components exist.
