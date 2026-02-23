# Systematic Debugging Checklist

When facing a production issue, follow this checklist methodically.

## Step 1: Gather Evidence

- [ ] Read complete error message
- [ ] Check browser console
- [ ] Check network tab (cookies sent?)
- [ ] Identify error type (401, 403, 500, etc.)
- [ ] Test if works locally
- [ ] Check recent deployments

## Step 2: Understand Architecture

- [ ] Read authentication docs
- [ ] Map data flow for failing feature
- [ ] Identify middleware vs API route behavior
- [ ] Understand token storage mechanism
- [ ] Review similar working features

## Step 3: Create Diagnostics

- [ ] Add debug endpoint if needed
- [ ] Add strategic logging
- [ ] Deploy diagnostics separately
- [ ] Test diagnostic endpoint
- [ ] Analyze diagnostic output

## Step 4: Identify Root Cause

- [ ] Use diagnostic data to pinpoint failure
- [ ] Verify with evidence (not assumptions)
- [ ] Trace data flow to failure point
- [ ] Check for type mismatches
- [ ] Verify environment differences

## Step 5: Implement Fix

- [ ] Fix root cause (not symptoms)
- [ ] One fix per deployment
- [ ] Write clear commit message
- [ ] Run lint and type-check
- [ ] Deploy and monitor

## Step 6: Verify Solution

- [ ] Test original failing case
- [ ] Test related functionality
- [ ] Check for regressions
- [ ] Update documentation if needed
- [ ] Remove temporary diagnostics (optional)

---

## Quick Reference: Error Types

| Error | Category | First Check |
|-------|----------|-------------|
| 401 | Authentication | Token present? Cookies sent? |
| 403 | Authorization | Permissions? Account access? |
| 404 | Routing | Path correct? API route exists? |
| 500 | Server | CloudWatch logs, stack trace |

## Quick Reference: Data Flow Points

```
Client Request
    |
    v
[1] Browser sends cookies? -> Check Network tab
    |
    v
[2] Middleware runs? -> Only for non-API routes
    |
    v
[3] Token extracted? -> Check extraction logic
    |
    v
[4] Claims parsed? -> Check JWT structure
    |
    v
[5] Database lookup? -> Check field names (cognitoId vs id)
    |
    v
[6] Permission check? -> Check role and permissions
    |
    v
Response
```

## Key Principles

1. **Diagnose Before Fix** - Understand the problem deeply before coding
2. **Evidence Over Assumptions** - Use diagnostics to verify theories
3. **One Change at a Time** - Incremental fixes are easier to debug
4. **Monitor Everything** - Track deployments to completion
5. **Security First** - Never bypass auth for convenience
6. **Document Learnings** - Update docs with new patterns
7. **Think Systematically** - Follow the data flow, find the break
