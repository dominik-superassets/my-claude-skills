---
name: linting-build-validator
description: Use when validating code before commits, fixing linting errors, checking TypeScript compilation, or ensuring build success. Runs ESLint, type-check, and production builds to catch issues early.
metadata:
  author: DDD
  version: 1.0.0
  category: validation
---

# Linting & Build Validator

Ensure 100% build success rate by catching and fixing issues before they reach production. This skill enforces zero-tolerance policy on linting errors, saving 20+ minutes per build failure.

## When to Use

- Before committing code to git
- Before creating pull requests
- Before deploying to production
- After making significant code changes
- When build is failing and you need diagnosis

## Critical Rules

1. **NO unused imports/variables** - Auto-fixable; variables can be prefixed with `_`
2. **NO `any` type or `@ts-ignore`** - Use `unknown` or specific types instead
3. **NO console.log or floating promises** - Use logger utilities; add `void` or `await`

## Core Commands

```bash
pnpm lint        # ESLint check
pnpm lint:fix    # Auto-fix linting issues
pnpm type-check  # TypeScript compilation
pnpm build       # Production build test
```

## Workflow Summary

1. Run `pnpm lint` and categorize errors
2. Apply auto-fixes with `pnpm lint:fix`
3. Run `pnpm type-check` for TypeScript validation
4. Run `pnpm build` to verify production build
5. Report issues with specific file/line references

## Reference Files

- `references/workflow.md` - Full validation workflow diagram
- `references/common-fixes.md` - Issue examples with code fixes
- `references/commands.md` - Complete command reference
- `references/checklist.md` - Pre-commit checklist and report format
