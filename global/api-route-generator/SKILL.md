---
name: api-route-generator
description: Create Next.js 15 API routes with authentication, permission checks, request validation, error handling, and database operations following SynthCore's established patterns. Use when creating api, new api route, generate endpoint, build api, add endpoint, create route, make api, scaffold api, implement endpoint, adding REST endpoints, building backend routes, implementing server actions, creating API handlers, or setting up data endpoints. Automatically generates routes with JWT verification, role-based access control, Zod validation schemas, proper Prisma usage, and comprehensive error responses.
---

# API Route Generator

## Overview

Generate production-ready Next.js 15 API routes that strictly follow SynthCore's backend patterns, saving **45+ minutes per endpoint** while ensuring security, performance, and consistency. This skill prevents critical mistakes like SQL injection, unauthorized access, connection pool exhaustion, and inconsistent error handling.

## When to Use This Skill

Use when:
- Creating new API endpoints for any resource
- Building CRUD operations (GET, POST, PATCH, DELETE)
- Adding authenticated routes
- Implementing admin-only endpoints
- Creating complex query endpoints with filtering/pagination
- Integrating external APIs or services
- Adding Deal Factory specific endpoints

## Workflow Decision Tree

```
User Request
    │
    ├─ Simple CRUD? → Basic Route Template
    │   └─ Auth needed? → Add JWT verification
    │       └─ Permissions? → Add role checks
    │
    ├─ Admin endpoint? → Admin Route Template
    │   └─ Audit logging? → Add AdminAuditLog
    │
    ├─ Complex query? → Advanced Query Template
    │   └─ Filtering? → Add Prisma where clauses
    │       └─ Pagination? → Add cursor/offset pagination
    │
    └─ External integration? → Integration Template
        └─ Rate limiting? → Add rate limit checks
            └─ Caching? → Add Redis/memory cache
```

## Core Capabilities

### 1. Route Structure Generation
- Next.js 15 App Router file-based routing
- HTTP method handlers (GET, POST, PATCH, DELETE, PUT)
- TypeScript types with NextRequest/NextResponse
- Async/await patterns with proper error boundaries
- Dynamic route segments `[id]`, `[...slug]`

### 2. Authentication & Authorization
**JWT Token Verification**:
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

**Role-Based Access Control**:
```typescript
// Admin-only check
if (user.role !== 'ADMIN' && user.role !== 'SUPER_ADMIN') {
  return NextResponse.json(
    { success: false, error: 'Admin access required' },
    { status: 403 }
  );
}

// Enterprise tier check
if (user.subscriptionTier !== 'ENTERPRISE') {
  return NextResponse.json(
    { success: false, error: 'Enterprise subscription required' },
    { status: 403 }
  );
}
```

**Deal Factory Profile Checks**:
```typescript
import { getInvestorProfile, getCompanyProfile } from '@/lib/deal-factory/profiles';

const investorProfile = await getInvestorProfile(user.id);
if (!investorProfile) {
  return NextResponse.json(
    { success: false, error: 'Investor profile required' },
    { status: 403 }
  );
}
```

### 3. Request Validation with Zod
**Query Parameter Validation**:
```typescript
import { z } from 'zod';

const querySchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  search: z.string().optional(),
  sortBy: z.enum(['name', 'createdAt', 'updatedAt']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc')
});

// Parse and validate
const { searchParams } = new URL(request.url);
const params = querySchema.parse(Object.fromEntries(searchParams));
```

**Request Body Validation**:
```typescript
const createResourceSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
  tags: z.array(z.string()).optional(),
  metadata: z.record(z.unknown()).optional()
});

const body = await request.json();
const validatedData = createResourceSchema.parse(body);
```

**Custom Validation Rules**:
```typescript
const dealSchema = z.object({
  amount: z.number().min(0),
  currency: z.enum(['USD', 'EUR', 'GBP']),
  stage: z.enum(['SEED', 'SERIES_A', 'SERIES_B', 'SERIES_C']),
  closeDate: z.string().datetime().refine(
    (date) => new Date(date) > new Date(),
    { message: 'Close date must be in the future' }
  )
});
```

### 4. Database Operations with Prisma
**CRITICAL: ALWAYS use shared Prisma client**:
```typescript
// ✅ CORRECT - Use shared client
import { prisma } from '@/lib/prisma';

// ❌ WRONG - Never create new instance
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient(); // Connection pool exhaustion!
```

**Optimized Queries**:
```typescript
// Use include for relations
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    portfolios: {
      include: {
        companies: {
          include: { company: true }
        }
      }
    }
  }
});

// Use select to limit fields
const companies = await prisma.company.findMany({
  select: {
    id: true,
    name: true,
    ticker: true,
    sector: true
  }
});
```

**Pagination Patterns**:
```typescript
// Offset-based pagination (simple)
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

// Cursor-based pagination (efficient for large datasets)
const items = await prisma.resource.findMany({
  where,
  take: limit + 1,
  cursor: cursor ? { id: cursor } : undefined,
  orderBy: { createdAt: 'desc' }
});

const hasNext = items.length > limit;
if (hasNext) items.pop();
```

**Transactions for Complex Operations**:
```typescript
const result = await prisma.$transaction(async (tx) => {
  // Create main resource
  const resource = await tx.resource.create({
    data: { name: 'Test' }
  });

  // Create related records
  await tx.relatedResource.createMany({
    data: relatedItems.map(item => ({
      resourceId: resource.id,
      ...item
    }))
  });

  // Update counts
  await tx.stats.update({
    where: { id: 'global' },
    data: { totalResources: { increment: 1 } }
  });

  return resource;
});
```

### 5. Error Handling Standards
**Comprehensive Try-Catch**:
```typescript
export async function POST(request: NextRequest) {
  try {
    // Route logic here

  } catch (error) {
    console.error('❌ API error:', error);

    // Validation errors (400)
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          success: false,
          error: 'Invalid request data',
          details: error.errors
        },
        { status: 400 }
      );
    }

    // Prisma unique constraint violations (409)
    if (error?.code === 'P2002') {
      return NextResponse.json(
        {
          success: false,
          error: 'Resource already exists',
          field: error.meta?.target
        },
        { status: 409 }
      );
    }

    // Prisma foreign key constraint (400)
    if (error?.code === 'P2003') {
      return NextResponse.json(
        {
          success: false,
          error: 'Related resource not found'
        },
        { status: 400 }
      );
    }

    // Generic error (500)
    return NextResponse.json(
      {
        success: false,
        error: error instanceof Error ? error.message : 'Internal server error'
      },
      { status: 500 }
    );
  }
}
```

**HTTP Status Codes**:
- `200` - Successful GET/PATCH
- `201` - Successful POST (resource created)
- `204` - Successful DELETE (no content)
- `400` - Bad request (validation error)
- `401` - Unauthorized (not authenticated)
- `403` - Forbidden (not authorized)
- `404` - Resource not found
- `409` - Conflict (duplicate resource)
- `429` - Too many requests (rate limit)
- `500` - Internal server error

## Step-by-Step Workflow

### Step 1: Analyze Request
Extract from user request:
- Resource name (e.g., "portfolios", "deals", "companies")
- Operations needed (GET list, GET by ID, POST create, PATCH update, DELETE)
- Authentication requirements (public, authenticated, admin-only)
- Permission requirements (role, subscription tier, profile type)
- Special features (pagination, filtering, sorting, search)
- External integrations (APIs, file uploads, webhooks)

### Step 2: Create Route File
```typescript
// File: app/api/[resource]/route.ts (for collection endpoints)
// File: app/api/[resource]/[id]/route.ts (for individual resources)

import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { getUserFromRequest } from '@/lib/auth/api-auth';
```

### Step 3: Define Validation Schemas
```typescript
// Query parameters for GET
const querySchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  search: z.string().optional()
});

// Request body for POST/PATCH
const createSchema = z.object({
  name: z.string().min(1).max(100),
  // ... other fields
});

const updateSchema = createSchema.partial(); // All fields optional
```

### Step 4: Implement GET Handler
```typescript
export async function GET(request: NextRequest) {
  try {
    // 1. Authentication (if required)
    const user = await getUserFromRequest(request);
    if (!user) {
      return NextResponse.json(
        { success: false, error: 'Authentication required' },
        { status: 401 }
      );
    }

    // 2. Validate query parameters
    const { searchParams } = new URL(request.url);
    const params = querySchema.parse(Object.fromEntries(searchParams));

    // 3. Build where clause
    const where = {
      userId: user.id, // Scope to user's data
      ...(params.search && {
        OR: [
          { name: { contains: params.search, mode: 'insensitive' } },
          { description: { contains: params.search, mode: 'insensitive' } }
        ]
      })
    };

    // 4. Fetch with pagination
    const skip = (params.page - 1) * params.limit;
    const [items, totalCount] = await Promise.all([
      prisma.resource.findMany({
        where,
        skip,
        take: params.limit,
        include: {
          // Include relations
        },
        orderBy: { createdAt: 'desc' }
      }),
      prisma.resource.count({ where })
    ]);

    // 5. Return success response
    return NextResponse.json({
      success: true,
      items,
      pagination: {
        page: params.page,
        limit: params.limit,
        totalCount,
        totalPages: Math.ceil(totalCount / params.limit)
      }
    });

  } catch (error) {
    // Error handling from Step 5
  }
}
```

### Step 5: Implement POST Handler
```typescript
export async function POST(request: NextRequest) {
  try {
    // 1. Authentication
    const user = await getUserFromRequest(request);
    if (!user) {
      return NextResponse.json(
        { success: false, error: 'Authentication required' },
        { status: 401 }
      );
    }

    // 2. Permission checks (if needed)
    if (requiresAdminAccess && user.role !== 'ADMIN') {
      return NextResponse.json(
        { success: false, error: 'Admin access required' },
        { status: 403 }
      );
    }

    // 3. Subscription limit checks (if applicable)
    const limitCheck = await checkSubscriptionLimit(user.id, 'resource');
    if (!limitCheck.allowed) {
      return NextResponse.json(
        {
          success: false,
          error: 'Resource limit reached',
          details: {
            limit: limitCheck.limit,
            used: limitCheck.used,
            tier: user.subscriptionTier
          }
        },
        { status: 403 }
      );
    }

    // 4. Validate request body
    const body = await request.json();
    const validatedData = createSchema.parse(body);

    // 5. Create resource
    const resource = await prisma.resource.create({
      data: {
        ...validatedData,
        userId: user.id
      },
      include: {
        // Include relations for response
      }
    });

    // 6. Log creation (for analytics/audit)
    console.log(`✅ Resource created: ${resource.id} by user ${user.id}`);

    // 7. Return success response with 201
    return NextResponse.json(
      {
        success: true,
        resource
      },
      { status: 201 }
    );

  } catch (error) {
    // Error handling
  }
}
```

### Step 6: Implement PATCH Handler (if needed)
```typescript
// File: app/api/[resource]/[id]/route.ts

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // 1. Authentication
    const user = await getUserFromRequest(request);
    if (!user) {
      return NextResponse.json(
        { success: false, error: 'Authentication required' },
        { status: 401 }
      );
    }

    // 2. Verify resource exists and user owns it
    const existing = await prisma.resource.findUnique({
      where: { id: params.id }
    });

    if (!existing) {
      return NextResponse.json(
        { success: false, error: 'Resource not found' },
        { status: 404 }
      );
    }

    if (existing.userId !== user.id && user.role !== 'ADMIN') {
      return NextResponse.json(
        { success: false, error: 'Access denied' },
        { status: 403 }
      );
    }

    // 3. Validate update data
    const body = await request.json();
    const validatedData = updateSchema.parse(body);

    // 4. Update resource
    const updated = await prisma.resource.update({
      where: { id: params.id },
      data: validatedData,
      include: {
        // Include relations
      }
    });

    return NextResponse.json({
      success: true,
      resource: updated
    });

  } catch (error) {
    // Error handling
  }
}
```

### Step 7: Implement DELETE Handler (if needed)
```typescript
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // Authentication and ownership check (same as PATCH)
    const user = await getUserFromRequest(request);
    // ... checks ...

    // Delete resource (use transaction if cascade deletes needed)
    await prisma.$transaction(async (tx) => {
      // Delete related records first
      await tx.relatedResource.deleteMany({
        where: { resourceId: params.id }
      });

      // Delete main resource
      await tx.resource.delete({
        where: { id: params.id }
      });
    });

    // Log deletion
    console.log(`✅ Resource deleted: ${params.id} by user ${user.id}`);

    return NextResponse.json(
      { success: true, message: 'Resource deleted' },
      { status: 200 }
    );

  } catch (error) {
    // Error handling
  }
}
```

## Reference Files

**Study these existing API routes for patterns before generating new ones**:

**Core Patterns**:
- `app/api/companies/route.ts` - Standard CRUD with pagination
- `app/api/admin/users/[id]/route.ts` - Admin-protected routes with audit logging
- `app/api/deal-factory/deals/route.ts` - Deal Factory specific patterns
- `app/api/enrichment/company/[id]/route.ts` - External API integration patterns

**Core Libraries**:
- `lib/prisma.ts` - **CRITICAL**: Shared Prisma client singleton (MUST use)
- `lib/auth/api-auth.ts` - JWT verification and auth utilities
- `lib/validation/deal-factory.ts` - Zod validation schema examples

**Documentation**:
- `CLAUDE.md` - Sections: "API Endpoints Reference", "Error Handling Patterns", "Database Operations"

## Common Patterns Reference

### Admin Endpoint with Audit Logging
```typescript
import { createAuditLog } from '@/lib/audit/logger';

// After admin action
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

### Complex Filtering
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
        validTo: null // Current scores only
      }
    }
  })
};
```

### Rate Limiting Check
```typescript
import { checkRateLimit } from '@/lib/rate-limit';

const rateLimitKey = `api:${user.id}:endpoint`;
const allowed = await checkRateLimit(rateLimitKey, 100, 3600); // 100 requests/hour

if (!allowed) {
  return NextResponse.json(
    { success: false, error: 'Rate limit exceeded' },
    { status: 429 }
  );
}
```

## Validation Checklist

Before delivering the API route, verify:
- [ ] Uses shared Prisma client from `@/lib/prisma`
- [ ] Authentication check present (if required)
- [ ] Permission/role checks implemented
- [ ] Request validation with Zod schemas
- [ ] Proper error handling with try-catch
- [ ] Correct HTTP status codes
- [ ] Consistent response format `{ success, data/error }`
- [ ] Pagination implemented for list endpoints
- [ ] Database queries optimized (no N+1 problems)
- [ ] Transactions used for multi-step operations
- [ ] Audit logging for admin actions
- [ ] TypeScript types defined for all parameters
- [ ] No console.logs in production code (use proper logging)

## Error Prevention

**CRITICAL MISTAKES TO AVOID**:
1. ❌ **Creating new PrismaClient()** → Use `import { prisma } from '@/lib/prisma'`
2. ❌ **Missing authentication checks** → Always verify user before data access
3. ❌ **No input validation** → Use Zod schemas for all inputs
4. ❌ **Inconsistent error responses** → Follow `{ success, error }` format
5. ❌ **Wrong status codes** → Use correct codes (401 auth, 403 permission, 404 not found)
6. ❌ **SQL injection risk** → Never use string interpolation, use Prisma parameterized queries
7. ❌ **N+1 queries** → Use `include` or `select` to fetch relations efficiently
8. ❌ **No pagination** → Always paginate list endpoints to prevent memory issues
9. ❌ **Missing transaction rollback** → Use `$transaction` for multi-step operations
10. ❌ **No audit logging** → Log all admin actions for compliance

## Resources

This skill references:
- **API Patterns**: Existing routes in `app/api/` directory
- **Authentication**: `lib/auth/api-auth.ts`
- **Database**: `lib/prisma.ts` (shared singleton)
- **Validation**: `lib/validation/` schemas
- **Documentation**: `CLAUDE.md` sections on API patterns
