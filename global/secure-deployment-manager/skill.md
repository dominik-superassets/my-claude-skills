---
name: secure-deployment-manager
description: Systematically deploy code changes to AWS Amplify with security-first debugging, comprehensive monitoring, and root cause analysis. Use when deploying branches, fixing production issues, troubleshooting authentication, debugging 401/403 errors, investigating deployment failures, monitoring Amplify jobs, analyzing auth flow, tracing token issues, or fixing database lookup errors. Prevents overengineering, ensures incremental fixes, and provides evidence-based debugging for Next.js 15 + AWS Cognito + Amplify deployments.
---

# Secure Deployment Manager

## Overview

Deploy code changes to AWS Amplify **systematically and securely** using evidence-based debugging, incremental deployment, and comprehensive monitoring. This skill captures the methodology from real production troubleshooting sessions, preventing hours of trial-and-error debugging through structured root cause analysis. Saves **3-5 hours per deployment issue** by avoiding common pitfalls like JWT overengineering, bundled fixes, and assumption-based debugging.

## When to Use This Skill

Use when:
- Deploying new features to AWS Amplify
- Fixing production 401/403 authentication errors
- Troubleshooting "works locally, fails on Amplify" issues
- Investigating deployment failures or build errors
- Debugging JWT/cookie authentication flows
- Monitoring multi-step deployment progress
- Analyzing Next.js middleware vs API route behavior
- Fixing database lookup mismatches (cognitoId vs id)
- Implementing diagnostic endpoints for production visibility
- Systematically debugging any production issue

---

## Core Philosophy

### 1. **Diagnose Before Fix**
- NEVER jump to solutions without understanding the root cause
- Evidence-based debugging: use logs, diagnostics, and debug endpoints
- Build diagnostic tools when existing logs are insufficient
- Verify assumptions with concrete data

### 2. **Incremental Deployment**
- One logical change per deployment
- Monitor each deployment to completion
- Verify functionality before proceeding
- Document what each job deploys

### 3. **Security-First**
- Authentication issues are architecture problems, not quick fixes
- Understand token flow: where stored, how transmitted, how validated
- Never bypass security for convenience
- Verify security in both dev and production environments

---

## Deployment Workflow

### Phase 1: Pre-Deployment Analysis

**Before writing any code:**

1. **Understand the Environment**
   ```bash
   # Check current deployment status
   aws amplify list-jobs --app-id <APP_ID> --branch-name <BRANCH> --max-items 3

   # Verify git state
   git status
   git log --oneline -5
   ```

2. **Identify the Problem**
   - Read error messages completely
   - Check browser console AND network tab
   - Distinguish between client-side and server-side errors
   - 401/403 = Authentication/Authorization
   - 500 = Server error (check server logs)
   - 404 = Routing issue

3. **Map the Architecture**
   - Where does authentication happen? (Middleware? API routes? Both?)
   - How are tokens stored? (Cookies? localStorage? Headers?)
   - What's the data flow? (Client → Auth → Database?)
   - Read relevant docs: `AUTHENTICATION_ARCHITECTURE.md`, `CLAUDE.md`

### Phase 2: Diagnostic Implementation

**When existing logs are insufficient:**

1. **Create Debug Endpoints** (for production visibility)
   ```typescript
   // app/api/debug/<feature>-check/route.ts
   export async function GET() {
     return NextResponse.json({
       success: true,
       debug: {
         timestamp: new Date().toISOString(),
         environment: process.env.NODE_ENV,
         // Return diagnostic info as JSON
         cookieCount: cookies.getAll().length,
         hasSomeToken: !!extractToken(cookies),
         // etc.
       }
     });
   }
   ```

2. **Add Strategic Logging** (for CloudWatch)
   ```typescript
   console.log('🔍 [FunctionName] Starting with:', { param1, param2 });
   console.log('✅ [FunctionName] Success:', result);
   console.error('❌ [FunctionName] Error:', error);
   ```

3. **Deploy Diagnostics First**
   - Commit diagnostic tools separately
   - Deploy and test diagnostics before attempting fixes
   - Use diagnostics to identify root cause

### Phase 3: Root Cause Analysis

**Use diagnostic data to find the real problem:**

1. **Follow the Data Flow**
   ```
   Where does the flow break?
   Client → [OK?] → Network → [OK?] → Server → [OK?] → Auth → [FAIL?] → Database
   ```

2. **Common Auth Issues**
   - **Cookies not sent**: Check cookie domain, SameSite, Secure flags
   - **Token not extracted**: Verify cookie name pattern, extraction logic
   - **Claims not parsed**: Check JWT structure, Base64 decoding
   - **Database lookup fails**: Verify field names (cognitoId vs id)
   - **Token expired**: Check exp claim vs current time

3. **Verify Assumptions**
   ```typescript
   // Don't assume - verify!
   console.log('Cookie type:', cookies?.constructor?.name);
   console.log('Has getAll:', typeof cookies?.getAll === 'function');
   console.log('Token found:', !!token);
   ```

### Phase 4: Targeted Fix

**Fix the root cause, not symptoms:**

1. **Single Responsibility**
   - One fix per deployment
   - Don't bundle unrelated changes
   - Keep git history clean

2. **Commit Message Format**
   ```
   fix|feat|refactor: brief description (50 chars max)

   Detailed explanation:
   - What was broken
   - Why it was broken
   - How this fixes it

   Root cause: <technical explanation>
   ```

3. **Pre-Commit Checks**
   ```bash
   pnpm lint
   pnpm type-check
   # Build only if critical (slow)
   ```

### Phase 5: Deployment Monitoring

**Track deployment through completion:**

1. **Monitor Job Progress**
   ```bash
   # Start background monitor
   sleep 480 && aws amplify get-job --app-id <APP> --branch-name <BRANCH> --job-id <ID>

   # Check detailed status
   aws amplify get-job --app-id <APP> --branch-name <BRANCH> --job-id <ID> \
     --query 'job.{summary:summary,steps:steps[*].{stepName:stepName,status:status}}'
   ```

2. **Track Multiple Deployments**
   - Use TodoWrite to track deployment jobs
   - Update status as jobs complete
   - Document what each job contains

3. **Verify Success**
   - Job status = SUCCEED
   - But also: **Test the actual functionality**
   - Don't assume success without verification

### Phase 6: Post-Deployment Verification

**Confirm the fix works:**

1. **Test the Original Failing Case**
   - Visit the URL that returned 401
   - Check browser console for errors
   - Verify expected behavior

2. **Test Related Functionality**
   - Don't just test the fixed endpoint
   - Test similar endpoints that use same auth flow
   - Verify no regressions

3. **Remove Diagnostic Code** (optional)
   - Keep debug endpoints if useful for future
   - Remove excessive console.logs from hot paths
   - Keep minimal logging for troubleshooting

---

## Authentication Troubleshooting Playbook

### Next.js 15 + AWS Amplify + Cognito Pattern

**Architecture:**
```
┌─────────────────────────────────────────────────────────┐
│                   AUTHENTICATION FLOW                    │
└─────────────────────────────────────────────────────────┘

User Login (Amplify Auth)
       ↓
JWT Tokens → Cookies (Amplify SSR: { ssr: true })
       ↓
       ├────────────────────┬─────────────────────┐
       ↓                    ↓                     ↓
  Page Routes          API Routes         Client Requests
  (/dashboard/*)       (/api/*)
       ↓                    ↓
  Middleware           Direct Cookie
  (sets headers)       Parsing (getAuthContext)
       ↓                    ↓
  x-user-role          JWT Claims Extraction
  x-user-tier          (parseTokenClaims)
       ↓                    ↓
       └────────────────────┴──────────────────────┐
                            ↓
                    Database Lookup
                    (by cognitoId, NOT id)
```

### Common Pitfalls

**1. Middleware vs API Route Confusion**
```typescript
// ❌ WRONG - Expecting middleware headers in API route
export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|public).*)']
};
// Middleware DOES NOT run for /api/* routes!

// ✅ CORRECT - API routes must parse cookies directly
export async function GET(request: NextRequest) {
  const context = await getAuthContext(request); // Handles both cases
}
```

**2. Database Lookup Field Mismatch**
```typescript
// ❌ WRONG - context.userId is Cognito ID (UUID)
const user = await prisma.user.findUnique({
  where: { id: context.userId } // id is Prisma CUID!
});

// ✅ CORRECT - Use cognitoId field
const user = await prisma.user.findUnique({
  where: { cognitoId: context.userId }
});
```

**3. Cookie Extraction Type Issues**
```typescript
// ❌ WRONG - Assuming one cookie type
function extractToken(cookies: Map<string, any>) {
  for (const [name, cookie] of cookies) { ... }
}

// ✅ CORRECT - Handle all Next.js cookie types
function extractToken(cookies: any) {
  // Handle RequestCookies (has getAll)
  if (typeof cookies.getAll === 'function') { ... }

  // Handle Map-like objects
  if (typeof cookies.entries === 'function') { ... }

  // Handle Headers
  if (cookies instanceof Headers) { ... }
}
```

**4. Development vs Production Environment**
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

---

## AWS Amplify Deployment Commands

### Quick Reference

```bash
# List recent deployments
aws amplify list-jobs \
  --app-id d1jv2latlju696 \
  --branch-name dev \
  --max-items 5 \
  --region us-east-1 \
  --profile synthcoreai-dev

# Get job details
aws amplify get-job \
  --app-id d1jv2latlju696 \
  --branch-name dev \
  --job-id <ID> \
  --region us-east-1 \
  --profile synthcoreai-dev

# Monitor job completion (background)
sleep 480 && aws amplify get-job \
  --app-id d1jv2latlju696 \
  --branch-name dev \
  --job-id <ID> \
  --region us-east-1 \
  --profile synthcoreai-dev \
  --query 'job.summary.{JobId:jobId,Status:status,EndTime:endTime}'
```

### Job Lifecycle

1. **PENDING** - Queued, waiting to start
2. **PROVISIONING** - Allocating resources
3. **RUNNING** - Building (see steps for detail)
4. **SUCCEED** - Completed successfully
5. **FAILED** - Build/deploy failed (check logs)

### Typical Build Times
- Simple code changes: 8-12 minutes
- Schema changes: 10-15 minutes
- Large refactors: 12-18 minutes

---

## Systematic Debugging Checklist

When facing a production issue:

### Step 1: Gather Evidence
- [ ] Read complete error message
- [ ] Check browser console
- [ ] Check network tab (cookies sent?)
- [ ] Identify error type (401, 403, 500, etc.)
- [ ] Test if works locally
- [ ] Check recent deployments

### Step 2: Understand Architecture
- [ ] Read authentication docs
- [ ] Map data flow for failing feature
- [ ] Identify middleware vs API route behavior
- [ ] Understand token storage mechanism
- [ ] Review similar working features

### Step 3: Create Diagnostics
- [ ] Add debug endpoint if needed
- [ ] Add strategic logging
- [ ] Deploy diagnostics separately
- [ ] Test diagnostic endpoint
- [ ] Analyze diagnostic output

### Step 4: Identify Root Cause
- [ ] Use diagnostic data to pinpoint failure
- [ ] Verify with evidence (not assumptions)
- [ ] Trace data flow to failure point
- [ ] Check for type mismatches
- [ ] Verify environment differences

### Step 5: Implement Fix
- [ ] Fix root cause (not symptoms)
- [ ] One fix per deployment
- [ ] Write clear commit message
- [ ] Run lint and type-check
- [ ] Deploy and monitor

### Step 6: Verify Solution
- [ ] Test original failing case
- [ ] Test related functionality
- [ ] Check for regressions
- [ ] Update documentation if needed
- [ ] Remove temporary diagnostics (optional)

---

## Real-World Example: Admin Dashboard 401 Fix

**Problem:** Admin dashboard worked locally but returned 401 on Amplify

**Wrong Approach (avoided):**
- ❌ "Just add JWT signature verification"
- ❌ Install jose library without understanding why
- ❌ Make multiple changes at once
- ❌ Assume it's an environment variable issue

**Correct Approach (used):**

1. **Diagnosed systematically**
   - Middleware doesn't run for /api/* routes (by design)
   - API routes need to parse cookies directly
   - getAuthContext only checked headers (failed for API routes)

2. **Fixed incrementally**
   - Job 39: Added cookie parsing to getAuthContext
   - Jobs 40-41: Added diagnostics (logging + debug endpoint)
   - Job 42: Fixed database lookup field (cognitoId vs id)

3. **Verified with evidence**
   - Debug endpoint showed cookies sent, token parsed, claims extracted
   - Revealed database lookup was using wrong field
   - Fixed and verified with actual test

**Result:** Three targeted fixes, each solving a specific issue, fully documented.

---

## Key Principles to Remember

1. **Diagnose Before Fix** - Understand the problem deeply before coding
2. **Evidence Over Assumptions** - Use diagnostics to verify theories
3. **One Change at a Time** - Incremental fixes are easier to debug
4. **Monitor Everything** - Track deployments to completion
5. **Security First** - Never bypass auth for convenience
6. **Document Learnings** - Update docs with new patterns
7. **Think Systematically** - Follow the data flow, find the break

---

**Skill Version:** 1.0
**Last Updated:** 2025-11-04
**Author:** SynthCore AI Development Team
**Based on:** Real production debugging session (Admin dashboard 401 fix)
