# Authentication Troubleshooting Playbook

## Next.js 15 + AWS Amplify + Cognito Pattern

### Architecture Overview

```
User Login (Amplify Auth)
       |
JWT Tokens -> Cookies (Amplify SSR: { ssr: true })
       |
       +--------------------+---------------------+
       |                    |                     |
  Page Routes          API Routes         Client Requests
  (/dashboard/*)       (/api/*)
       |                    |
  Middleware           Direct Cookie
  (sets headers)       Parsing (getAuthContext)
       |                    |
  x-user-role          JWT Claims Extraction
  x-user-tier          (parseTokenClaims)
       |                    |
       +--------------------+----------------------+
                            |
                    Database Lookup
                    (by cognitoId, NOT id)
```

## Common Pitfalls

### 1. Middleware vs API Route Confusion

```typescript
// WRONG - Expecting middleware headers in API route
export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|public).*)']
};
// Middleware DOES NOT run for /api/* routes!

// CORRECT - API routes must parse cookies directly
export async function GET(request: NextRequest) {
  const context = await getAuthContext(request); // Handles both cases
}
```

### 2. Database Lookup Field Mismatch

```typescript
// WRONG - context.userId is Cognito ID (UUID)
const user = await prisma.user.findUnique({
  where: { id: context.userId } // id is Prisma CUID!
});

// CORRECT - Use cognitoId field
const user = await prisma.user.findUnique({
  where: { cognitoId: context.userId }
});
```

### 3. Cookie Extraction Type Issues

```typescript
// WRONG - Assuming one cookie type
function extractToken(cookies: Map<string, any>) {
  for (const [name, cookie] of cookies) { ... }
}

// CORRECT - Handle all Next.js cookie types
function extractToken(cookies: any) {
  // Handle RequestCookies (has getAll)
  if (typeof cookies.getAll === 'function') { ... }

  // Handle Map-like objects
  if (typeof cookies.entries === 'function') { ... }

  // Handle Headers
  if (cookies instanceof Headers) { ... }
}
```

### 4. Development vs Production Environment

```typescript
// Local works, Amplify fails = Environment difference!

// Check:
if (process.env.NODE_ENV === 'development') {
  // Dev bypass may be active locally
  return SUPER_ADMIN_CONTEXT;
}

// Production requires real auth
// No bypasses should exist in production
```

## Diagnostic Strategies

### Create Debug Endpoints

```typescript
// app/api/debug/auth-check/route.ts
export async function GET(request: NextRequest) {
  const cookies = request.cookies;

  return NextResponse.json({
    success: true,
    debug: {
      timestamp: new Date().toISOString(),
      environment: process.env.NODE_ENV,
      cookieCount: cookies.getAll().length,
      cookieNames: cookies.getAll().map(c => c.name),
      hasCognitoToken: cookies.getAll().some(c =>
        c.name.includes('CognitoIdentityServiceProvider')
      ),
    }
  });
}
```

### Token Extraction Debugging

```typescript
console.log('Cookie type:', cookies?.constructor?.name);
console.log('Has getAll:', typeof cookies?.getAll === 'function');
console.log('Cookie names:', cookies.getAll?.().map(c => c.name));
console.log('Token found:', !!token);
console.log('Token claims:', claims ? Object.keys(claims) : 'none');
```

## Quick Diagnosis Flow

1. **401 Unauthorized**
   - Check: Are cookies being sent? (Network tab)
   - Check: Is token being extracted? (Debug endpoint)
   - Check: Are claims being parsed? (Server logs)
   - Check: Is database lookup succeeding? (Server logs)

2. **403 Forbidden**
   - Check: Is user authenticated? (Token present)
   - Check: Does user have required permission?
   - Check: Is account hierarchy check passing?

3. **500 Server Error**
   - Check: CloudWatch logs for stack trace
   - Check: Database connection
   - Check: Environment variables

## Environment Checklist

- [ ] Cognito User Pool ID configured
- [ ] Cognito Client ID configured
- [ ] Cookie domain matches deployment domain
- [ ] SameSite and Secure flags appropriate for environment
- [ ] Database connection string correct
- [ ] All environment variables present in Amplify
