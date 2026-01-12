# Feature Implementation Template

Use this template for implementing new features with the Ralph Wiggum loop.

## Template

```markdown
# Feature: [FEATURE_NAME]

## Context
[Brief description of what this feature does and why it's needed]

## Requirements
- [ ] [Requirement 1]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

## Implementation Phases

### Phase 1: [Component/Foundation]
**Goal:** [Specific deliverable]
**Files to create/modify:**
- `path/to/file.ts`
- `path/to/test.ts`

**Steps:**
1. Write failing tests for [specific behavior]
2. Run tests - verify they fail
3. Implement minimal code to pass
4. Run tests - verify they pass
5. Commit with message: "feat: [description]"

### Phase 2: [Next Component]
**Goal:** [Specific deliverable]
**Files to create/modify:**
- `path/to/file.ts`

**Steps:**
1. Write failing tests for [specific behavior]
2. Run tests - verify they fail
3. Implement minimal code to pass
4. Run tests - verify they pass
5. Commit with message: "feat: [description]"

[Add more phases as needed]

## Completion Criteria
- [ ] All requirements implemented
- [ ] All tests passing (coverage > 80%)
- [ ] Linter passes with no errors
- [ ] Documentation updated
- [ ] Code follows project conventions

## Verification Commands
```bash
npm test
npm run lint
npm run build
```

When ALL criteria are met and ALL verification commands pass, output:
<promise>COMPLETE</promise>

## Self-Correction Protocol
If any test fails or verification command fails:
1. Read the error message carefully
2. Identify the root cause
3. Fix the issue
4. Re-run verification
5. Repeat until all pass
6. NEVER output COMPLETE until ALL verifications pass
```

## Good Example

```markdown
# Feature: User Authentication JWT

## Context
Add JWT-based authentication to the REST API for secure user sessions.

## Requirements
- [ ] Login endpoint returns JWT token
- [ ] Protected routes validate JWT
- [ ] Token expiration handled (1 hour)
- [ ] Refresh token mechanism
- [ ] Logout invalidates token

## Implementation Phases

### Phase 1: JWT Token Generation
**Goal:** Login endpoint that returns valid JWT
**Files:**
- `src/auth/jwt.service.ts`
- `src/auth/auth.controller.ts`
- `tests/auth/jwt.service.test.ts`

**Steps:**
1. Write test: `login returns JWT for valid credentials`
2. Run: `npm test -- jwt.service` - verify FAIL
3. Implement JwtService.generateToken()
4. Run: `npm test -- jwt.service` - verify PASS
5. Commit: "feat(auth): add JWT token generation"

### Phase 2: Route Protection Middleware
**Goal:** Middleware that validates JWT on protected routes
**Files:**
- `src/middleware/auth.middleware.ts`
- `tests/middleware/auth.middleware.test.ts`

**Steps:**
1. Write test: `middleware rejects requests without token`
2. Write test: `middleware rejects expired tokens`
3. Write test: `middleware accepts valid tokens`
4. Run tests - verify FAIL
5. Implement AuthMiddleware
6. Run tests - verify PASS
7. Commit: "feat(auth): add JWT validation middleware"

### Phase 3: Refresh Token
**Goal:** Refresh token endpoint and rotation
**Files:**
- `src/auth/refresh.service.ts`
- `tests/auth/refresh.service.test.ts`

**Steps:**
1. Write test: `refresh returns new access token`
2. Write test: `refresh invalidates old token`
3. Run tests - verify FAIL
4. Implement RefreshService
5. Run tests - verify PASS
6. Commit: "feat(auth): add refresh token rotation"

## Completion Criteria
- [ ] All 5 requirements implemented
- [ ] All tests passing (coverage > 80%)
- [ ] No linter errors
- [ ] API docs updated with auth endpoints

## Verification Commands
```bash
npm test -- --coverage
npm run lint
npm run build
```

When ALL criteria met and ALL commands pass, output:
<promise>COMPLETE</promise>

## Self-Correction Protocol
If any test fails:
1. Read error message
2. Identify if it's implementation bug or test bug
3. Fix the actual issue (not the test unless test is wrong)
4. Re-run verification
5. Repeat until green
```
