# Command Reference

## Core Validation Commands

```bash
# Full validation suite
pnpm lint        # ESLint check
pnpm type-check  # TypeScript compilation
pnpm build       # Production build test

# Auto-fix commands
pnpm lint:fix    # Auto-fix linting issues
pnpm format      # Format with Prettier

# Combined validation (recommended before commit)
pnpm ci          # Runs lint + type-check + build
```

## Validation Steps

### Step 1: Run Linting Check
```bash
pnpm lint
```
Parse output to identify:
- Unused imports
- Unused variables
- Console.log statements
- Floating promises
- Formatting issues
- Custom rule violations

### Step 2: Apply Auto-Fixes
```bash
pnpm lint:fix  # Applies ESLint auto-fixes
pnpm format    # Applies Prettier formatting
```

### Step 3: TypeScript Validation
```bash
pnpm type-check  # Equivalent to: tsc --noEmit
```

### Step 4: Build Simulation
```bash
pnpm build  # Full Next.js production build
```

## Git Integration

### Pre-Commit Hook Example
Create `.husky/pre-commit`:
```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run validation before commit
pnpm lint
pnpm type-check

# If validation fails, prevent commit
if [ $? -ne 0 ]; then
  echo "Validation failed. Please fix errors before committing."
  exit 1
fi
```

### Manual Pre-Commit Check
Before every commit, ask Claude:
- "Check if my code is ready to commit"
- "Validate code before deployment"
- "Run pre-commit checks"

## Configuration Files

Read these to understand validation rules:
- `eslint.config.js` - ESLint rules and configuration
- `tsconfig.json` - TypeScript compiler options (strict mode)
- `next.config.js` - Next.js build configuration
- `.prettierrc` - Code formatting rules

## Package Scripts Reference

From `package.json`:
- `lint` - Run ESLint
- `lint:fix` - Run ESLint with auto-fix
- `type-check` - TypeScript compilation check
- `build` - Production build
- `format` - Prettier formatting
- `ci` - Combined lint + type-check + build

## Tools Used

- **ESLint 9** with Next.js and TypeScript plugins
- **TypeScript 5** with strict mode enabled
- **Next.js 15** build system
- **Prettier 3** for consistent formatting
