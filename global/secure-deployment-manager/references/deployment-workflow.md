# Deployment Workflow - 6 Phases

Systematic deployment process for AWS Amplify with evidence-based debugging.

## Phase 1: Pre-Deployment Analysis

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
   - What's the data flow? (Client -> Auth -> Database?)
   - Read relevant docs: `AUTHENTICATION_ARCHITECTURE.md`, `CLAUDE.md`

## Phase 2: Diagnostic Implementation

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
   console.log('[FunctionName] Starting with:', { param1, param2 });
   console.log('[FunctionName] Success:', result);
   console.error('[FunctionName] Error:', error);
   ```

3. **Deploy Diagnostics First**
   - Commit diagnostic tools separately
   - Deploy and test diagnostics before attempting fixes
   - Use diagnostics to identify root cause

## Phase 3: Root Cause Analysis

**Use diagnostic data to find the real problem:**

1. **Follow the Data Flow**
   ```
   Where does the flow break?
   Client -> [OK?] -> Network -> [OK?] -> Server -> [OK?] -> Auth -> [FAIL?] -> Database
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

## Phase 4: Targeted Fix

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

## Phase 5: Deployment Monitoring

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

## Phase 6: Post-Deployment Verification

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

## Job Lifecycle

1. **PENDING** - Queued, waiting to start
2. **PROVISIONING** - Allocating resources
3. **RUNNING** - Building (see steps for detail)
4. **SUCCEED** - Completed successfully
5. **FAILED** - Build/deploy failed (check logs)

## Typical Build Times

- Simple code changes: 8-12 minutes
- Schema changes: 10-15 minutes
- Large refactors: 12-18 minutes
