# Prompt Writing Rules

## 1. Atomic Steps

Each step = one action (2-5 minutes work):

**Bad:**
```
Implement the authentication system with JWT tokens and refresh logic
```

**Good:**
```
Phase 1: JWT Generation
Step 1: Write test for generateToken()
Step 2: Run test - verify FAIL
Step 3: Implement generateToken()
Step 4: Run test - verify PASS
Step 5: Commit
```

## 2. Explicit Verification

Every phase must have:
- Specific commands to run
- Expected outcomes
- What failure looks like

**Bad:**
```
Make sure it works
```

**Good:**
```
Run: npm test -- auth.test.ts
Expected: 5 tests, 5 passing
If fails: Check error message, fix implementation, re-run
```

## 3. Clear Completion Signal

Always use the exact promise format:
```markdown
When ALL criteria met, output:
<promise>COMPLETE</promise>
```

Include fallback for stuck scenarios:
```markdown
After 15 failed attempts at same step:
Document blockers in docs/debug/[task].md
Output: <promise>BLOCKED</promise>
```

## 4. Self-Correction Patterns

Include explicit fix loops:

```markdown
## Self-Correction Protocol
If any test fails:
1. DO NOT skip or modify test (unless test is wrong)
2. Read full error message
3. Trace to root cause in implementation
4. Apply minimal fix
5. Re-run ALL tests
6. If still failing, repeat from step 2
7. After 5 attempts at same error, try different approach
```

## 5. Context Independence

Prompt must work without conversation history:
- Include all file paths
- Specify exact commands
- Define all terms
- No "as discussed" or "like before"
