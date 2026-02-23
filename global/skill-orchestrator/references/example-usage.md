# Skill Orchestrator - Example Usage

## Example 1: Single Skill Injection

**User request:** "Create an API endpoint for user preferences"

**Orchestrator reasoning:**
1. Task involves API creation → `api-route-generator` skill applies
2. Read the skill, compose prompt, spawn agent

**Orchestrator actions:**

```
Step 1: Read skill file
Tool: Read
Path: global/api-route-generator/SKILL.md
Result: {skill content}

Step 2: Compose prompt
---
You are a specialized autonomous agent with expert knowledge.

<skill-knowledge name="api-route-generator">
# API Route Generator

## Overview
Generate production-ready Next.js 15 API routes...
{full skill content}
</skill-knowledge>

---

## Your Task

<task>
Create an API endpoint for user preferences with:
- GET: Retrieve user's preferences
- PATCH: Update preferences
- Proper authentication
- Zod validation
</task>

<context>
- Project: Next.js 15 with Prisma
- Auth: NextAuth v5 with JWT
- Location: src/app/api/preferences/route.ts
</context>

<output-requirements>
Return JSON: { status, result, artifacts, issues, request_skills, spawn_requests }
</output-requirements>
---

Step 3: Spawn Task
Tool: Task
subagent_type: general-purpose
model: sonnet
prompt: {composed prompt above}

Step 4: Process result
Agent returns:
{
  "status": "complete",
  "result": "Created preferences API with GET and PATCH handlers",
  "artifacts": ["src/app/api/preferences/route.ts"],
  "issues": [],
  "request_skills": ["linting-build-validator"],
  "spawn_requests": []
}

Step 5: Chain to validator (since skill chains_to linting-build-validator)
Read linting-build-validator skill, spawn validation agent
```

---

## Example 2: Multi-Skill Parallel Execution

**User request:** "Build a complete settings feature with API and UI"

**Orchestrator reasoning:**
1. Needs backend → `api-route-generator`
2. Needs frontend → `component-generator`
3. Both can run in parallel
4. Then validate with `linting-build-validator`

**Orchestrator actions:**

```
Step 1: Read both skills in parallel
Tool: Read (api-route-generator/SKILL.md)
Tool: Read (component-generator/SKILL.md)

Step 2: Spawn parallel agents (single message with multiple Task calls)

Task 1 (Backend):
---
<skill-knowledge name="api-route-generator">
{content}
</skill-knowledge>

<task>Create settings API endpoint</task>
---

Task 2 (Frontend):
---
<skill-knowledge name="component-generator">
{content}
</skill-knowledge>

<task>Create SettingsForm component</task>
---

Step 3: Collect results
Agent 1: { status: "complete", artifacts: ["src/app/api/settings/route.ts"] }
Agent 2: { status: "complete", artifacts: ["src/components/settings-form.tsx"] }

Step 4: Validate all artifacts
Read linting-build-validator, spawn with all artifacts

Step 5: Report to user
"Created settings feature:
- API: src/app/api/settings/route.ts
- Component: src/components/settings-form.tsx
- Validation: passed"
```

---

## Example 3: Recursive Delegation with Refinement

**User request:** "Create a dashboard widget component"

```
Step 1: Spawn component-generator agent
Result: {
  "status": "needs_refinement",
  "result": "Created DashboardWidget but needs design review",
  "artifacts": ["src/components/dashboard-widget.tsx"],
  "request_skills": ["design-system-auditor"]
}

Step 2: Orchestrator sees needs_refinement + request_skills
Read design-system-auditor skill

Step 3: Spawn refinement agent with BOTH skills
---
<skill-knowledge name="component-generator">
{content}
</skill-knowledge>

<skill-knowledge name="design-system-auditor">
{content}
</skill-knowledge>

<previous-result>
{JSON from step 1}
</previous-result>

<refinement-task>
Apply design-system-auditor patterns to improve the DashboardWidget.
Check: dark mode, spacing, colors, accessibility
</refinement-task>
---

Result: {
  "status": "complete",
  "result": "Refined DashboardWidget with design system compliance",
  "artifacts": ["src/components/dashboard-widget.tsx"],
  "fixes_applied": [
    "Added dark mode variants",
    "Fixed spacing to use design tokens",
    "Added aria labels"
  ]
}

Step 4: Validate and report
```

---

## Example 4: Workflow Execution

**User request:** "Run the full-feature workflow for notifications"

**Orchestrator checks skill-mesh.json:**
```json
"full-feature": {
  "skills": ["api-route-generator", "component-generator", "linting-build-validator"],
  "execution": "parallel-then-validate"
}
```

**Execution:**
1. Spawn api-route-generator agent (notifications API)
2. Spawn component-generator agent (NotificationCenter component) [parallel]
3. Wait for both to complete
4. Spawn linting-build-validator with all artifacts
5. Report complete feature

---

## Example 5: Blocked Agent Escalation

```
Agent returns:
{
  "status": "blocked",
  "result": "Cannot create API - Prisma schema missing User model",
  "issues": [
    "User model not found in prisma/schema.prisma",
    "Need schema update before API creation"
  ],
  "spawn_requests": [
    {
      "task": "Add User model to Prisma schema",
      "suggested_skills": [],
      "context": "Required before notifications API can be created",
      "priority": "high"
    }
  ]
}

Orchestrator:
1. Reports blocker to user
2. "The notifications API requires a User model in Prisma schema. Should I:
   A) Create the User model first
   B) Provide the schema for you to review
   C) Skip this and proceed differently"
3. Wait for user decision
```

---

## Key Patterns

### Pattern: Always Chain to Validator
```
After any code-generating skill:
→ Read linting-build-validator
→ Spawn validation agent
→ Only report "complete" after validation passes
```

### Pattern: Combine Related Skills
```
API + UI tasks → inject both api-route-generator AND component-generator
AWS tasks → inject aws-infrastructure-manager AND aws-cdk-development
```

### Pattern: Escalate, Don't Retry Blindly
```
If agent returns "blocked" or fails twice:
→ Don't retry with same approach
→ Report to user with context
→ Ask for guidance
```

### Pattern: Preserve Context Chain
```
Each agent receives:
→ Skill knowledge (from SKILL.md)
→ Task (specific to this agent)
→ Context (from parent, including previous results)
→ Output contract (structured response format)
```
