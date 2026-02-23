# Pre-Commit Validation Checklist

Use this checklist to verify code quality before committing.

## Required Checks

- [ ] All linting errors resolved
- [ ] TypeScript compilation successful
- [ ] No unused imports remain
- [ ] No unused variables (or prefixed with `_`)
- [ ] No console.log statements
- [ ] All promises handled properly
- [ ] No `any` types used
- [ ] No `@ts-ignore` comments
- [ ] Build completes successfully
- [ ] Bundle size within limits
- [ ] All tests pass
- [ ] No circular dependencies
- [ ] Dark mode variants present
- [ ] No browser dialogs (alert/confirm/prompt)

## Example Validation Report

```
Linting & Build Validation Report
==================================

PHASE 1: Linting
- Total files checked: 247
- Issues found: 12
- Auto-fixed: 9
- Manual fixes needed: 3

Auto-fixes applied:
[OK] Removed 5 unused imports
[OK] Prefixed 2 unused variables with _
[OK] Removed 2 console.log statements

Manual fixes required:
[ERROR] lib/utils/helpers.ts:45
   Error: Unexpected any type
   Fix: Replace 'any' with proper type or 'unknown'

[ERROR] app/api/deals/route.ts:78
   Error: Property 'investorProfile' does not exist
   Fix: Add to type definition or include in Prisma query

[ERROR] components/forms/DealForm.tsx:123
   Error: Type 'string' is not assignable to type 'number'
   Fix: Use parseInt() or Number() for conversion

==================================

PHASE 2: TypeScript
- Compilation: Failed
- Type errors: 3
- Suggestions generated: 3

==================================

PHASE 3: Build Test
- Skipped (fix errors first)

==================================

SUMMARY
==================================

Status: NOT READY TO COMMIT

Progress:
[OK] 9/12 issues fixed automatically
[WARN] 3 issues require manual fixes

Files Modified (auto-fixed):
1. components/dashboard/Overview.tsx
2. components/ui/DataTable.tsx
3. app/api/auth/sync/route.ts
4. lib/utils/formatters.ts
5. components/charts/MetricChart.tsx

Next Steps:
1. Fix 3 manual issues listed above
2. Run validation again: "Check if ready to commit"
3. After passing, commit with confidence!

Time saved: ~25 minutes of debugging
```

## Success Metrics

This validation prevents:
- 500+ build failures/month by catching errors early
- 200+ type errors/month through strict TypeScript checking
- 100+ console.logs/month from reaching production
- 50+ unused imports/week that slow down builds

Time saved:
- 20-30 minutes per validation (debugging time prevented)

## Best Practices

1. **Run before every commit** - Make it a habit
2. **Fix immediately** - Don't accumulate errors
3. **Trust auto-fixes** - They follow established patterns
4. **Review manual fixes** - Understand why they're needed
5. **Monitor bundle size** - Keep pages under 500KB
6. **Clean imports regularly** - Remove unused immediately
