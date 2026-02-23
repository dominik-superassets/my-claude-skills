# API Route Template

## Standard Route Structure

```typescript
// File: app/api/[resource]/route.ts (collection endpoints)
// File: app/api/[resource]/[id]/route.ts (individual resources)

import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { getUserFromRequest } from '@/lib/auth/api-auth';
```

## GET Handler (List)

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
      userId: user.id,
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
    // See error-handling.md for error patterns
  }
}
```

## POST Handler (Create)

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

    // 3. Validate request body
    const body = await request.json();
    const validatedData = createSchema.parse(body);

    // 4. Create resource
    const resource = await prisma.resource.create({
      data: {
        ...validatedData,
        userId: user.id
      }
    });

    // 5. Return success response with 201
    return NextResponse.json(
      { success: true, resource },
      { status: 201 }
    );

  } catch (error) {
    // See error-handling.md for error patterns
  }
}
```

## PATCH Handler (Update)

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
      data: validatedData
    });

    return NextResponse.json({
      success: true,
      resource: updated
    });

  } catch (error) {
    // See error-handling.md for error patterns
  }
}
```

## DELETE Handler

```typescript
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await getUserFromRequest(request);
    // ... authentication and ownership checks ...

    // Delete with transaction if cascade needed
    await prisma.$transaction(async (tx) => {
      await tx.relatedResource.deleteMany({
        where: { resourceId: params.id }
      });
      await tx.resource.delete({
        where: { id: params.id }
      });
    });

    return NextResponse.json(
      { success: true, message: 'Resource deleted' },
      { status: 200 }
    );

  } catch (error) {
    // See error-handling.md for error patterns
  }
}
```
