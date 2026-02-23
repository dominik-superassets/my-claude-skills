---
name: api-route-generator
description: Use when creating Next.js 15 API routes, REST endpoints, or backend handlers. Triggers: new api, generate endpoint, build api, add endpoint, create route, scaffold api, implement endpoint, adding backend routes.
metadata:
  author: DDD
  version: 1.0.0
  category: backend
---

# API Route Generator

## Overview

Generate production-ready Next.js 15 API routes with authentication, permissions, Zod validation, and Prisma database operations. Prevents critical security issues (SQL injection, unauthorized access) and ensures consistent patterns across all endpoints.

## When to Use

- Creating new API endpoints for any resource
- Building CRUD operations (GET, POST, PATCH, DELETE)
- Adding authenticated or admin-only routes
- Implementing pagination, filtering, or search
- Integrating external APIs or services

## Core Pattern

Every API route follows this sequence:

1. **Authentication** - Verify JWT via `getUserFromRequest()`
2. **Authorization** - Check role/permissions
3. **Validation** - Parse inputs with Zod schemas
4. **Database** - Use shared `prisma` client (NEVER `new PrismaClient()`)
5. **Response** - Return `{ success, data/error }` with correct status code

Key imports:
```typescript
import { prisma } from '@/lib/prisma';
import { getUserFromRequest } from '@/lib/auth/api-auth';
```

## Quick Reference

| Operation | Status Code |
|-----------|-------------|
| GET success | 200 |
| POST create | 201 |
| DELETE | 200/204 |
| Validation error | 400 |
| Unauthorized | 401 |
| Forbidden | 403 |
| Not found | 404 |

## References

- `references/route-template.md` - Full GET/POST/PATCH/DELETE templates
- `references/patterns.md` - CRUD, pagination, filtering, transactions
- `references/validation-checklist.md` - Pre-commit verification checklist
- `references/error-handling.md` - Error patterns and Prisma codes

## Troubleshooting

### 401 Unauthorized on all routes

**Cause**: `getUserFromRequest()` can't extract the JWT from the request. Common when Authorization header is stripped by a proxy (nginx, Amplify).

**Fix**: Check that the auth middleware reads from the correct source (cookie vs header). For Amplify/nginx deployments, ensure the mu-plugin or proxy config forwards the Authorization header.

### Prisma connection errors in production

**Cause**: Connection pool exhaustion from `new PrismaClient()` being called per-request instead of using the shared singleton.

**Fix**: ALWAYS import from `@/lib/prisma` — never instantiate `new PrismaClient()`. The shared client manages connection pooling.

### Zod validation passes but database rejects data

**Cause**: Zod schema allows values that violate database constraints (unique, enum, length limits).

**Fix**: Align Zod schemas with Prisma schema constraints. Use `.max()` for string length limits, `.refine()` for uniqueness checks that need a database query, and match enum values exactly.

### Route works in dev but 404 in production

**Cause**: File-based routing path doesn't match the expected URL. Common with Next.js 15 App Router `route.ts` vs Pages Router `pages/api/`.

**Fix**: Verify the route file is in the correct directory for your router type. App Router: `app/api/{path}/route.ts`. Pages Router: `pages/api/{path}.ts`. Check that the HTTP method export matches (`GET`, `POST` etc.).

## Chain Handoff

After generating API routes, chain to validation:

1. **linting-build-validator** — Run `pnpm lint` and `pnpm type-check` to verify the route compiles and follows linting rules

When orchestrated by skill-orchestrator or used in a pipeline:
- Return the list of created/modified file paths so downstream validators know what to check
- DO NOT run validation yourself — let the orchestrator chain the appropriate validation skill
- If generating multiple routes, they can be validated as a batch after all generation is complete
