# Common Violation Patterns

## Pattern 1: Success Colors

```typescript
// WRONG - Green for general actions
<Button className="bg-green-600">Save</Button>

// CORRECT - Blue/purple for primary, green only for semantic success
<Button variant="primary">Save</Button>  // Uses blue/purple gradient
<Badge variant="success">Completed</Badge>  // Green only for status
```

## Pattern 2: Tab Navigation

```typescript
// WRONG - Green active tabs
className={activeTab ? "border-green-500 text-green-600" : "..."}

// CORRECT - Blue/purple active tabs
className={activeTab ? "border-blue-500 text-blue-600 dark:text-blue-400" : "..."}
```

## Pattern 3: Interactive Cards

```typescript
// WRONG - Missing hover effects
<Card className="p-6">...</Card>

// CORRECT - Full hover treatment
<Card
  neonBorder
  interactive
  className="p-6 cursor-pointer hover:shadow-xl hover:-translate-y-1 transition-all duration-300"
>
  ...
</Card>
```

## Pattern 4: Browser Dialog Replacements

```typescript
// WRONG - Using alert for notifications
alert('Settings saved successfully');

// CORRECT - Using toast notifications
import { useToast } from '@/components/ui/toast';
const { toast } = useToast();
toast({ title: 'Settings saved successfully', variant: 'success' });
```

```typescript
// WRONG - Using confirm for destructive actions
if (confirm('Are you sure you want to delete this?')) {
  deleteItem();
}

// CORRECT - Using Modal component
import { Modal } from '@/components/ui/Modal';
import { ConfirmDialog } from '@/components/ui/ConfirmDialog';

<Modal open={showConfirm} onOpenChange={setShowConfirm}>
  <ConfirmDialog
    title="Delete Item"
    description="Are you sure you want to delete this? This action cannot be undone."
    onConfirm={deleteItem}
    onCancel={() => setShowConfirm(false)}
  />
</Modal>
```

```typescript
// WRONG - Using prompt for user input
const name = prompt('Enter your name:');

// CORRECT - Using Modal with form
<Modal open={showInput} onOpenChange={setShowInput}>
  <form onSubmit={handleSubmit}>
    <Input label="Name" value={name} onChange={setName} />
    <Button type="submit">Submit</Button>
  </form>
</Modal>
```

## Pattern 5: Missing Dark Mode

```typescript
// WRONG - No dark mode variants
<div className="bg-white border-gray-200 text-gray-900">
  <p className="text-gray-600">Description</p>
</div>

// CORRECT - Complete dark mode support
<div className="bg-white dark:bg-gray-800 border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white">
  <p className="text-gray-600 dark:text-gray-400">Description</p>
</div>
```

## Pattern 6: Raw HTML Elements

```typescript
// WRONG - Raw button element
<button className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
  Save
</button>

// CORRECT - Design system Button
import { Button } from '@/components/ui/Button';
<Button variant="primary">Save</Button>
```

```typescript
// WRONG - Raw div as card
<div className="bg-white rounded-xl shadow-sm border p-6">
  Card content
</div>

// CORRECT - Design system Card
import { Card } from '@/components/ui/Card';
<Card className="p-6">Card content</Card>
```

```typescript
// WRONG - Raw span as badge
<span className="px-2 py-1 text-xs rounded-full bg-green-100 text-green-800">
  Active
</span>

// CORRECT - Design system Badge
import { Badge } from '@/components/ui/Badge';
<Badge variant="success">Active</Badge>
```

## Pattern 7: Non-Standard Spacing

```typescript
// WRONG - Non-standard padding
<div className="p-5">...</div>  // p-5 is non-standard
<div className="p-7">...</div>  // p-7 is non-standard

// CORRECT - Standard padding values
<div className="p-4">...</div>  // Small
<div className="p-6">...</div>  // Medium (default)
<div className="p-8">...</div>  // Large
```

```typescript
// WRONG - Non-standard gap
<div className="flex gap-4">...</div>  // gap-4 is non-standard
<div className="flex gap-5">...</div>  // gap-5 is non-standard

// CORRECT - Standard gap values
<div className="flex gap-2">...</div>  // Tight
<div className="flex gap-3">...</div>  // Small
<div className="flex gap-6">...</div>  // Medium (default)
```

## Pattern 8: Missing Hover Transitions

```typescript
// WRONG - Hover without transition
<button className="bg-blue-600 hover:bg-blue-700">
  Click
</button>

// CORRECT - Smooth hover transition
<button className="bg-blue-600 hover:bg-blue-700 transition-all duration-200">
  Click
</button>
```

```typescript
// WRONG - Card without hover effects
<Card className="p-6">Content</Card>

// CORRECT - Card with hover effects
<Card className="p-6 hover:shadow-xl hover:-translate-y-1 transition-all duration-300">
  Content
</Card>
```

## Pattern 9: Gradient Inconsistencies

```typescript
// WRONG - Inconsistent gradient direction
className="bg-gradient-to-l from-blue-600 to-purple-600"  // Left direction

// CORRECT - Standard right direction
className="bg-gradient-to-r from-blue-600 to-purple-600"  // Right direction
```

```typescript
// WRONG - Missing hover gradient
className="bg-gradient-to-r from-blue-600 to-purple-600"

// CORRECT - With hover state
className="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700"
```

## Pattern 10: Shadow Color Mismatch

```typescript
// WRONG - Default shadow
className="shadow-lg"

// CORRECT - Themed shadow with color
className="shadow-lg shadow-blue-500/20"  // Blue glow effect
className="shadow-xl shadow-purple-500/20"  // Purple glow effect
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
