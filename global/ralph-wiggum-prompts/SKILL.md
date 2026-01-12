---
name: ralph-wiggum-prompts
description: Transform plans, tasks, features, and requests into properly structured prompts for the Ralph Wiggum autonomous loop. Use when preparing work for `/ralph-loop:ralph-loop`, when structuring tasks for iterative autonomous execution, or when user mentions "ralph", "wiggum", "loop prompt", or wants to automate re-prompting. Ensures clear completion criteria, incremental phases, self-correction patterns, and proper `<promise>` markers.
---

# Ralph Wiggum Prompt Engineering

Transform user plans, tasks, features, and requests into prompts optimized for the Ralph Wiggum autonomous loop.

**Announce:** "Using ralph-wiggum-prompts skill to structure this for autonomous execution."

## Ralph Wiggum Loop Overview

The loop repeatedly feeds Claude a prompt until completion:
```bash
/ralph-loop:ralph-loop "$(cat docs/ralph-prompts/task.md)" --completion-promise "COMPLETE" --max-iterations 10
```

**Critical insight:** Success depends on prompt quality, not model capability. Prompts must be:
- Self-contained with clear completion criteria
- Incrementally structured (phases/steps)
- Self-correcting (test, verify, fix, repeat)
- Explicit about the completion signal

## The Transformation Process

### Step 1: Analyze Input

Identify what user provided:
- **Plan document** - already structured, needs loop adaptation
- **Feature request** - needs full structuring
- **Bug report** - needs investigation + fix structure
- **Refactoring goal** - needs safety-first structure
- **Vague request** - needs clarification first

Ask clarifying questions if input is ambiguous:
- "What does 'done' look like for this task?"
- "What verification commands should pass?"
- "Are there specific files/components involved?"

### Step 2: Select Template

Choose based on task type:

| Task Type | Template | Key Pattern |
|-----------|----------|-------------|
| New feature | [feature-template.md](references/feature-template.md) | Phased implementation |
| TDD work | [tdd-template.md](references/tdd-template.md) | Red-Green-Refactor cycles |
| Bug fix | [bugfix-template.md](references/bugfix-template.md) | Reproduce-Fix-Verify |
| Refactoring | [refactor-template.md](references/refactor-template.md) | Safety-first incremental |

### Step 3: Structure the Prompt

Every Ralph prompt MUST have these sections:

```markdown
# [Task Type]: [Clear Title]

## Context
[Brief background - what, why, where in codebase]

## Requirements
- [ ] [Checkable requirement 1]
- [ ] [Checkable requirement 2]

## Implementation Phases

### Phase 1: [Name]
**Goal:** [Specific deliverable]
**Files:** [Exact paths]
**Steps:**
1. [Atomic action]
2. [Verification step]
3. [Commit step]

### Phase 2: [Name]
[Repeat pattern]

## Completion Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All tests pass
- [ ] Linter clean

## Verification Commands
\`\`\`bash
[exact commands to run]
\`\`\`

When ALL criteria met AND ALL commands pass, output:
<promise>COMPLETE</promise>

## Self-Correction Protocol
If verification fails:
1. Read error carefully
2. Identify root cause
3. Fix issue
4. Re-run verification
5. Repeat until ALL pass
6. NEVER output COMPLETE until verified
```

## Prompt Writing Rules

### 1. Atomic Steps
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

### 2. Explicit Verification

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

### 3. Clear Completion Signal

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

### 4. Self-Correction Patterns

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

### 5. Context Independence

Prompt must work without conversation history:
- Include all file paths
- Specify exact commands
- Define all terms
- No "as discussed" or "like before"

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Vague goals | Loop never completes | Add checkable criteria |
| Missing verification | Can't detect success | Add explicit test commands |
| Giant steps | Overwhelms single iteration | Break into atomic actions |
| No self-correction | Gets stuck on errors | Add fix-and-retry protocol |
| Implicit completion | Unclear when done | Use `<promise>` tags |

## Output Format

**CRITICAL: Always write to file.** Multi-line prompts fail with inline commands due to newline parsing.

### Step 1: Create Prompt Directory
```bash
mkdir -p docs/ralph-prompts
```

### Step 2: Write Prompt to File

Use the Write tool to save the structured prompt to: `docs/ralph-prompts/<task-name>.md`

Naming convention: kebab-case, descriptive (e.g., `fix-static-assets.md`, `add-user-auth.md`)

### Step 3: Output Ready-to-Run Command

After writing the file, output:

```bash
/ralph-loop:ralph-loop "$(cat docs/ralph-prompts/<task-name>.md)" --max-iterations <N> --completion-promise "<promise-text>"
```

### Step 4: Provide Summary

- **Prompt file:** `docs/ralph-prompts/<task-name>.md`
- **Max iterations:** (10-20 depending on complexity)
- **Watch for:** task-specific failure points

### Complete Example

For a bug fix request, Claude should:

1. **Create the prompt file** using Write tool:
   ```
   docs/ralph-prompts/fix-static-assets.md
   ```
   (Contains full structured prompt with phases, verification, self-correction)

2. **Output the command:**
   ```bash
   /ralph-loop:ralph-loop "$(cat docs/ralph-prompts/fix-static-assets.md)" --max-iterations 15 --completion-promise "Choo choo! Ralph fixed it!"
   ```

3. **Summary:**
   > Prompt saved to `docs/ralph-prompts/fix-static-assets.md`
   > Recommended: 15 iterations (debugging task with deployment)
   > Watch for: Docker rebuild needed after file changes

## When NOT Suitable for Ralph

Redirect user if task requires:
- Subjective human judgment
- Interactive user decisions mid-task
- Unclear success criteria that can't be defined
- Production debugging with real data
- Tasks needing external approvals

Suggest alternatives:
- Manual implementation with superpowers:test-driven-development
- Interactive session with superpowers:brainstorming first
