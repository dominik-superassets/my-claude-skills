# TDD Development Template

Use this template for strict test-driven development with the Ralph Wiggum loop.

## Template

```markdown
# TDD: [COMPONENT_NAME]

## Objective
[What this component/feature should do]

## Red-Green-Refactor Cycle

### Cycle 1: [First Behavior]

**RED - Write Failing Test**
```[language]
test('[descriptive name]', () => {
  // Arrange
  const input = [setup]

  // Act
  const result = functionUnderTest(input)

  // Assert
  expect(result).toBe(expected)
})
```

**Verify RED**
Run: `[test command]`
Expected: FAIL with "[expected error message]"

**GREEN - Minimal Implementation**
```[language]
function functionUnderTest(input) {
  // Minimal code to pass test
  return expected
}
```

**Verify GREEN**
Run: `[test command]`
Expected: PASS

**Commit**
```bash
git add .
git commit -m "feat: [what this cycle added]"
```

### Cycle 2: [Next Behavior]
[Repeat RED-GREEN pattern]

### Cycle 3: [Edge Case]
[Repeat RED-GREEN pattern]

## Refactor Phase
After all behaviors implemented:
- Remove duplication
- Improve names
- Extract helpers
- Keep tests green throughout

Run: `[test command]`
Expected: ALL PASS

Commit: `git commit -m "refactor: [what was improved]"`

## Completion Criteria
- [ ] All behaviors have tests
- [ ] Each test was seen failing before passing
- [ ] All tests pass
- [ ] Code refactored and clean
- [ ] No linter warnings

## Verification
```bash
[full test suite command]
[lint command]
```

When ALL pass, output:
<promise>COMPLETE</promise>

## Self-Correction
If test fails unexpectedly:
1. Check if test is correct (tests behavior, not implementation)
2. Check if implementation matches spec
3. Fix implementation (not test) unless test is wrong
4. Re-run, repeat until green
```

## Good Example

```markdown
# TDD: Email Validator

## Objective
Create email validation utility that handles common formats and edge cases.

## Red-Green-Refactor Cycle

### Cycle 1: Valid Simple Email

**RED - Write Failing Test**
```typescript
test('accepts valid simple email', () => {
  expect(isValidEmail('user@example.com')).toBe(true)
})
```

**Verify RED**
Run: `npm test -- email.test.ts`
Expected: FAIL with "isValidEmail is not defined"

**GREEN - Minimal Implementation**
```typescript
export function isValidEmail(email: string): boolean {
  return email.includes('@') && email.includes('.')
}
```

**Verify GREEN**
Run: `npm test -- email.test.ts`
Expected: PASS

**Commit**
```bash
git add src/email.ts tests/email.test.ts
git commit -m "feat: add basic email validation"
```

### Cycle 2: Reject Empty String

**RED - Write Failing Test**
```typescript
test('rejects empty string', () => {
  expect(isValidEmail('')).toBe(false)
})
```

**Verify RED**
Run: `npm test -- email.test.ts`
Expected: FAIL (empty string passes current implementation)

**GREEN - Update Implementation**
```typescript
export function isValidEmail(email: string): boolean {
  if (!email) return false
  return email.includes('@') && email.includes('.')
}
```

**Verify GREEN**
Run: `npm test -- email.test.ts`
Expected: ALL PASS

**Commit**
```bash
git commit -am "feat: reject empty email"
```

### Cycle 3: Reject Missing Domain

**RED - Write Failing Test**
```typescript
test('rejects email without domain', () => {
  expect(isValidEmail('user@')).toBe(false)
})
```

**Verify RED**
Run: `npm test -- email.test.ts`
Expected: FAIL

**GREEN - Update Implementation**
```typescript
export function isValidEmail(email: string): boolean {
  if (!email) return false
  const [local, domain] = email.split('@')
  return local && domain && domain.includes('.')
}
```

**Verify GREEN**
Run: `npm test -- email.test.ts`
Expected: ALL PASS

**Commit**
```bash
git commit -am "feat: validate email domain"
```

### Cycle 4: Reject Multiple @ Signs

**RED**
```typescript
test('rejects multiple @ signs', () => {
  expect(isValidEmail('user@@example.com')).toBe(false)
})
```

**Verify RED** - Expected: FAIL

**GREEN**
```typescript
export function isValidEmail(email: string): boolean {
  if (!email) return false
  const parts = email.split('@')
  if (parts.length !== 2) return false
  const [local, domain] = parts
  return local && domain && domain.includes('.')
}
```

**Verify GREEN** - Expected: ALL PASS
**Commit**: `git commit -am "feat: reject multiple @ signs"`

## Refactor Phase
After all cycles:
- Extract regex for better readability
- Add JSDoc comments
- Keep tests green

```typescript
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

export function isValidEmail(email: string): boolean {
  return EMAIL_REGEX.test(email)
}
```

Run: `npm test -- email.test.ts`
Expected: ALL PASS

Commit: `git commit -am "refactor: use regex for email validation"`

## Completion Criteria
- [x] Valid email accepted
- [x] Empty string rejected
- [x] Missing domain rejected
- [x] Multiple @ rejected
- [x] All tests pass
- [x] Code refactored

## Verification
```bash
npm test -- email.test.ts --coverage
npm run lint
```

When ALL pass, output:
<promise>COMPLETE</promise>
```
