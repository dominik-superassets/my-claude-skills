# Common API Patterns

## Validation Schemas

### Query Parameters
```typescript
const querySchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  search: z.string().optional(),
  sortBy: z.enum(['name', 'createdAt', 'updatedAt']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc')
});
```

### Request Body
```typescript
const createSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
  tags: z.array(z.string()).optional(),
  metadata: z.record(z.unknown()).optional()
});

const updateSchema = createSchema.partial();
```

### Custom Validation
```typescript
const dealSchema = z.object({
  amount: z.number().min(0),
  currency: z.enum(['USD', 'EUR', 'GBP']),
  closeDate: z.string().datetime().refine(
    (date) => new Date(date) > new Date(),
    { message: 'Close date must be in the future' }
  )
});
```

## Pagination

### Offset-Based (Simple)
```typescript
const skip = (page - 1) * limit;
const [items, totalCount] = await Promise.all([
  prisma.resource.findMany({
    where,
    skip,
    take: limit,
    orderBy: { createdAt: 'desc' }
  }),
  prisma.resource.count({ where })
]);
```

### Cursor-Based (Large Datasets)
```typescript
const items = await prisma.resource.findMany({
  where,
  take: limit + 1,
  cursor: cursor ? { id: cursor } : undefined,
  orderBy: { createdAt: 'desc' }
});

const hasNext = items.length > limit;
if (hasNext) items.pop();
```

## Complex Filtering
```typescript
import { Prisma } from '@prisma/client';

const where: Prisma.CompanyWhereInput = {
  ...(params.search && {
    OR: [
      { name: { contains: params.search, mode: 'insensitive' } },
      { ticker: { contains: params.search, mode: 'insensitive' } }
    ]
  }),
  ...(params.sector && { sector: params.sector }),
  ...(params.minScore !== undefined && {
    esgMetrics: {
      some: {
        totalScore: { gte: params.minScore },
        validTo: null
      }
    }
  })
};
```

## Transactions
```typescript
const result = await prisma.$transaction(async (tx) => {
  const resource = await tx.resource.create({
    data: { name: 'Test' }
  });

  await tx.relatedResource.createMany({
    data: relatedItems.map(item => ({
      resourceId: resource.id,
      ...item
    }))
  });

  await tx.stats.update({
    where: { id: 'global' },
    data: { totalResources: { increment: 1 } }
  });

  return resource;
});
```

## Admin Audit Logging
```typescript
await tx.adminAuditLog.create({
  data: {
    adminId: admin.id,
    action: 'UPDATE_RESOURCE',
    resource: 'resource',
    resourceId: resource.id,
    changeDetails: {
      before: previousState,
      after: newState,
      reason: 'Admin updated resource'
    },
    ipAddress: request.headers.get('x-forwarded-for') || 'unknown'
  }
});
```

## Rate Limiting
```typescript
import { checkRateLimit } from '@/lib/rate-limit';

const rateLimitKey = `api:${user.id}:endpoint`;
const allowed = await checkRateLimit(rateLimitKey, 100, 3600);

if (!allowed) {
  return NextResponse.json(
    { success: false, error: 'Rate limit exceeded' },
    { status: 429 }
  );
}
```

## Authentication Patterns

### JWT Verification
```typescript
import { getUserFromRequest } from '@/lib/auth/api-auth';

const user = await getUserFromRequest(request);
if (!user) {
  return NextResponse.json(
    { success: false, error: 'Authentication required' },
    { status: 401 }
  );
}
```

### Role-Based Access
```typescript
if (user.role !== 'ADMIN' && user.role !== 'SUPER_ADMIN') {
  return NextResponse.json(
    { success: false, error: 'Admin access required' },
    { status: 403 }
  );
}
```

### Subscription Tier Check
```typescript
if (user.subscriptionTier !== 'ENTERPRISE') {
  return NextResponse.json(
    { success: false, error: 'Enterprise subscription required' },
    { status: 403 }
  );
}
```

## Optimized Queries

### Include Relations
```typescript
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    portfolios: {
      include: {
        companies: { include: { company: true } }
      }
    }
  }
});
```

### Select Specific Fields
```typescript
const companies = await prisma.company.findMany({
  select: {
    id: true,
    name: true,
    ticker: true,
    sector: true
  }
});
```
