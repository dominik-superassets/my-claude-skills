# API Route Validation Checklist

## Pre-Delivery Verification

Before delivering any API route, verify ALL items:

### Database
- [ ] Uses shared Prisma client: `import { prisma } from '@/lib/prisma'`
- [ ] NO `new PrismaClient()` anywhere in the code
- [ ] Database queries optimized (no N+1 problems)
- [ ] Uses `include` or `select` for relations
- [ ] Transactions used for multi-step operations

### Authentication & Authorization
- [ ] Authentication check present (if required)
- [ ] Permission/role checks implemented
- [ ] Resource ownership verified before updates/deletes
- [ ] Admin actions require admin role check

### Validation
- [ ] Request validation with Zod schemas
- [ ] Query parameters validated
- [ ] Request body validated
- [ ] TypeScript types defined for all parameters

### Error Handling
- [ ] Proper try-catch wrapping
- [ ] Correct HTTP status codes used
- [ ] Consistent response format: `{ success, data/error }`
- [ ] Zod errors return 400 with details
- [ ] Prisma errors handled appropriately

### Performance
- [ ] Pagination implemented for list endpoints
- [ ] No unbounded queries (always use `take`)
- [ ] Efficient filtering with Prisma `where`

### Logging & Audit
- [ ] Audit logging for admin actions
- [ ] No console.logs in production code
- [ ] Errors logged with context

## HTTP Status Codes Reference

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate resource |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected error |

## Critical Mistakes to Avoid

1. **Creating new PrismaClient()** - Always use shared client
2. **Missing authentication checks** - Always verify user first
3. **No input validation** - Use Zod for all inputs
4. **Inconsistent error responses** - Follow `{ success, error }` format
5. **Wrong status codes** - Match code to error type
6. **SQL injection risk** - Use Prisma parameterized queries only
7. **N+1 queries** - Use `include`/`select` for relations
8. **No pagination** - Always paginate list endpoints
9. **Missing transaction rollback** - Use `$transaction` for multi-step
10. **No audit logging** - Log all admin actions
