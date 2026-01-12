# Bug Fix Template

Use this template for fixing bugs with the Ralph Wiggum loop.

## Template

```markdown
# Bug Fix: [BUG_TITLE]

## Bug Description
**Reported behavior:** [What's happening]
**Expected behavior:** [What should happen]
**Reproduction steps:**
1. [Step 1]
2. [Step 2]
3. [Observe bug]

## Investigation Phase

### Step 1: Reproduce the Bug
```bash
[command to reproduce]
```
Expected output: [buggy behavior description]

### Step 2: Write Failing Test
Create a test that captures the exact bug:
```[language]
test('[bug description as test name]', () => {
  // Setup that triggers bug
  const result = buggyFunction(triggerInput)

  // Assert expected (correct) behavior
  expect(result).toBe(correctBehavior)
})
```

### Step 3: Verify Test Fails
Run: `[test command]`
Expected: FAIL with current (buggy) behavior

## Fix Phase

### Step 4: Identify Root Cause
Location: `path/to/file.ts:line_number`
Cause: [Explain why the bug occurs]

### Step 5: Implement Fix
```[language]
// Before (buggy)
[buggy code]

// After (fixed)
[fixed code]
```

### Step 6: Verify Fix
Run: `[test command]`
Expected: PASS

### Step 7: Check for Regressions
Run: `[full test suite]`
Expected: ALL PASS (no new failures)

### Step 8: Commit
```bash
git add .
git commit -m "fix: [concise description of fix]"
```

## Completion Criteria
- [ ] Bug reproduced with failing test
- [ ] Root cause identified
- [ ] Fix implemented
- [ ] Regression test passes
- [ ] No other tests broken
- [ ] No linter errors

## Verification
```bash
[test command]
[lint command]
```

When ALL criteria met, output:
<promise>COMPLETE</promise>

## Self-Correction Protocol
If fix breaks other tests:
1. DO NOT modify other tests to pass
2. Understand why they broke
3. Revise fix to not break existing behavior
4. Re-run all tests
5. Repeat until ALL green

After 15 failed attempts:
1. Document findings in `docs/debug/[bug-name].md`
2. List attempted fixes and why they failed
3. Output: <promise>BLOCKED - SEE DEBUG DOC</promise>
```

## Good Example

```markdown
# Bug Fix: Login Accepts Empty Password

## Bug Description
**Reported behavior:** Users can login with empty password
**Expected behavior:** Empty password should be rejected with error
**Reproduction steps:**
1. Go to /login
2. Enter valid username
3. Leave password empty
4. Click submit
5. User is logged in (BUG!)

## Investigation Phase

### Step 1: Reproduce the Bug
```bash
curl -X POST /api/login -d '{"username":"admin","password":""}'
```
Expected output: `{"success":true,"token":"..."}` (BUG - should fail)

### Step 2: Write Failing Test
```typescript
test('rejects login with empty password', async () => {
  const response = await request(app)
    .post('/api/login')
    .send({ username: 'validuser', password: '' })

  expect(response.status).toBe(401)
  expect(response.body.error).toBe('Password required')
})
```

### Step 3: Verify Test Fails
Run: `npm test -- login.test.ts`
Expected: FAIL - receives 200 instead of 401

## Fix Phase

### Step 4: Identify Root Cause
Location: `src/auth/login.controller.ts:45`
Cause: Validation only checks if password field exists, not if it's non-empty

```typescript
// Current (buggy) code
if (password === undefined) {
  return res.status(401).json({ error: 'Password required' })
}
```

### Step 5: Implement Fix
```typescript
// Before (buggy)
if (password === undefined) {
  return res.status(401).json({ error: 'Password required' })
}

// After (fixed)
if (!password || password.trim() === '') {
  return res.status(401).json({ error: 'Password required' })
}
```

### Step 6: Verify Fix
Run: `npm test -- login.test.ts`
Expected: PASS

### Step 7: Check for Regressions
Run: `npm test`
Expected: ALL PASS

### Step 8: Commit
```bash
git add src/auth/login.controller.ts tests/auth/login.test.ts
git commit -m "fix(auth): reject login with empty password"
```

## Completion Criteria
- [x] Bug reproduced with failing test
- [x] Root cause: missing empty string check
- [x] Fix: check for falsy and whitespace-only
- [x] Regression test passes
- [x] All other tests pass
- [x] Linter clean

## Verification
```bash
npm test
npm run lint
```

When ALL pass, output:
<promise>COMPLETE</promise>
```
