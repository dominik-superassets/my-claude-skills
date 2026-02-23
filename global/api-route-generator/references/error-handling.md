# Error Handling Patterns

## Comprehensive Try-Catch Template

```typescript
export async function POST(request: NextRequest) {
  try {
    // Route logic here

  } catch (error) {
    console.error('API error:', error);

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

    // Prisma record not found (404)
    if (error?.code === 'P2025') {
      return NextResponse.json(
        {
          success: false,
          error: 'Resource not found'
        },
        { status: 404 }
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

## Common Prisma Error Codes

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| P2002 | Unique constraint violation | 409 Conflict |
| P2003 | Foreign key constraint failed | 400 Bad Request |
| P2025 | Record not found | 404 Not Found |
| P2014 | Required relation violation | 400 Bad Request |

## Authentication Errors

```typescript
// Not authenticated
if (!user) {
  return NextResponse.json(
    { success: false, error: 'Authentication required' },
    { status: 401 }
  );
}
```

## Authorization Errors

```typescript
// Not authorized (role)
if (user.role !== 'ADMIN') {
  return NextResponse.json(
    { success: false, error: 'Admin access required' },
    { status: 403 }
  );
}

// Not authorized (ownership)
if (resource.userId !== user.id) {
  return NextResponse.json(
    { success: false, error: 'Access denied' },
    { status: 403 }
  );
}
```

## Resource Not Found

```typescript
const resource = await prisma.resource.findUnique({
  where: { id: params.id }
});

if (!resource) {
  return NextResponse.json(
    { success: false, error: 'Resource not found' },
    { status: 404 }
  );
}
```

## Validation Errors

```typescript
// Zod validation
try {
  const validatedData = schema.parse(body);
} catch (error) {
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
}
```

## Rate Limit Errors

```typescript
if (!allowed) {
  return NextResponse.json(
    { success: false, error: 'Rate limit exceeded' },
    { status: 429 }
  );
}
```

## Subscription Limit Errors

```typescript
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
```

## Response Format Standard

All responses MUST follow this format:

### Success Response
```typescript
{
  success: true,
  data: { ... },
  // Optional: pagination, metadata
}
```

### Error Response
```typescript
{
  success: false,
  error: 'Human readable message',
  // Optional: details, field, code
}
```
