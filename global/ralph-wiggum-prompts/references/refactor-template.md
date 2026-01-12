# Refactoring Template

Use this template for safe refactoring with the Ralph Wiggum loop.

## Template

```markdown
# Refactor: [REFACTORING_NAME]

## Objective
[What improvement this refactoring achieves - readability, performance, maintainability]

## Safety Constraints
- NO behavior changes
- ALL existing tests must pass throughout
- If tests break, revert and understand why

## Pre-Refactor Verification

### Step 1: Ensure Clean Baseline
```bash
[run full test suite]
[run linter]
```
Expected: ALL PASS with no warnings

If not passing, FIX FIRST before refactoring.

### Step 2: Document Current Behavior
Files being refactored:
- `path/to/file1.ts` - [what it does]
- `path/to/file2.ts` - [what it does]

## Incremental Refactoring

### Change 1: [First Small Change]
**What:** [Description]
**Why:** [Benefit]

```[language]
// Before
[old code]

// After
[new code]
```

**Verify:**
```bash
[test command]
```
Expected: ALL PASS

**Commit:**
```bash
git commit -am "refactor: [what changed]"
```

### Change 2: [Second Small Change]
[Repeat pattern]

### Change 3: [Third Small Change]
[Repeat pattern]

## Post-Refactor Verification

### Final Test Run
```bash
[full test suite]
[linter]
[type check if applicable]
```
Expected: ALL PASS

### Behavior Verification
Confirm same behavior:
```bash
[command to verify behavior unchanged]
```

## Completion Criteria
- [ ] All existing tests pass
- [ ] No behavior changes
- [ ] Code is cleaner/faster/more maintainable
- [ ] Each change committed separately
- [ ] No linter warnings

When ALL criteria met, output:
<promise>COMPLETE</promise>

## Self-Correction Protocol
If ANY test fails after a change:
1. IMMEDIATELY revert: `git checkout -- .`
2. Understand why the change broke tests
3. Either:
   a. The refactoring changed behavior (bad) - rethink approach
   b. The test is too coupled to implementation - note but don't fix during refactor
4. Try smaller incremental change
5. Re-run tests after each tiny step
```

## Good Example

```markdown
# Refactor: Extract User Validation Logic

## Objective
Move scattered validation logic into dedicated UserValidator class for better testability and reuse.

## Safety Constraints
- NO behavior changes
- ALL existing tests must pass throughout
- If tests break, revert and understand why

## Pre-Refactor Verification

### Step 1: Ensure Clean Baseline
```bash
npm test
npm run lint
```
Expected: ALL PASS (127 tests, 0 failures)

### Step 2: Document Current Behavior
Files being refactored:
- `src/controllers/user.controller.ts` - has inline validation
- `src/controllers/profile.controller.ts` - duplicates validation
- `src/services/signup.service.ts` - more duplication

## Incremental Refactoring

### Change 1: Create Empty Validator Class
**What:** Create UserValidator with no methods
**Why:** Establish structure first

```typescript
// src/validators/user.validator.ts (NEW)
export class UserValidator {
  // Methods will be added incrementally
}
```

**Verify:**
```bash
npm test
```
Expected: ALL PASS (new file, no behavior change)

**Commit:**
```bash
git add src/validators/user.validator.ts
git commit -m "refactor: add empty UserValidator class"
```

### Change 2: Extract Email Validation
**What:** Move email validation to UserValidator
**Why:** Single source of truth for email rules

```typescript
// src/validators/user.validator.ts
export class UserValidator {
  static isValidEmail(email: string): boolean {
    // Exact same logic from user.controller.ts:45
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }
}

// src/controllers/user.controller.ts
// Before
if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { ... }

// After
if (!UserValidator.isValidEmail(email)) { ... }
```

**Verify:**
```bash
npm test
```
Expected: ALL PASS

**Commit:**
```bash
git commit -am "refactor: extract email validation to UserValidator"
```

### Change 3: Extract Password Validation
**What:** Move password strength check to UserValidator
**Why:** Consistent password rules across app

```typescript
// src/validators/user.validator.ts
static isValidPassword(password: string): boolean {
  return password.length >= 8 && /\d/.test(password)
}
```

**Verify:** `npm test` - ALL PASS
**Commit:** `git commit -am "refactor: extract password validation"`

### Change 4: Update Profile Controller
**What:** Replace duplicated validation in profile.controller.ts
**Why:** Remove duplication

```typescript
// Before
if (email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { ... }

// After
if (email && !UserValidator.isValidEmail(email)) { ... }
```

**Verify:** `npm test` - ALL PASS
**Commit:** `git commit -am "refactor: use UserValidator in profile controller"`

### Change 5: Update Signup Service
**What:** Replace duplicated validation in signup.service.ts
**Why:** Complete the consolidation

```typescript
// Replace all inline validation with UserValidator calls
```

**Verify:** `npm test` - ALL PASS
**Commit:** `git commit -am "refactor: use UserValidator in signup service"`

## Post-Refactor Verification

### Final Test Run
```bash
npm test
npm run lint
npm run typecheck
```
Expected: ALL PASS (still 127 tests)

### Behavior Verification
```bash
# Same validation behavior
curl -X POST /api/users -d '{"email":"invalid"}' # Should fail
curl -X POST /api/users -d '{"email":"valid@test.com","password":"short"}' # Should fail
curl -X POST /api/users -d '{"email":"valid@test.com","password":"validpass1"}' # Should succeed
```

## Completion Criteria
- [x] All 127 tests still pass
- [x] No behavior changes (same responses)
- [x] Validation now in single UserValidator class
- [x] 5 incremental commits
- [x] No linter warnings

When ALL pass, output:
<promise>COMPLETE</promise>
```
