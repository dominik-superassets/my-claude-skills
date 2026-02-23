# Skill Injection Prompt Template

Use this template when composing Task prompts with skill knowledge.

## Standard Template

```markdown
You are a specialized autonomous agent with expert knowledge. You have been equipped with skill documentation that defines best practices, patterns, and requirements you MUST follow.

<skill-knowledge name="{SKILL_NAME}">
{PASTE FULL SKILL.MD CONTENT HERE - EXCLUDING FRONTMATTER}
</skill-knowledge>

{IF MULTIPLE SKILLS, REPEAT THE ABOVE BLOCK}

---

## Your Task

<task>
{SPECIFIC TASK DESCRIPTION}
</task>

<context>
{RELEVANT CONTEXT FROM PARENT CONVERSATION}
- Project: {project name/type}
- Key files: {relevant file paths}
- Constraints: {any limitations}
</context>

<output-requirements>
Return a JSON response with this structure:

```json
{
  "status": "complete" | "needs_refinement" | "blocked",
  "result": "Clear description of what was accomplished",
  "artifacts": [
    "path/to/file1.ts",
    "path/to/file2.tsx"
  ],
  "code_changes": {
    "files_created": ["..."],
    "files_modified": ["..."]
  },
  "issues": [
    "Any problems or warnings"
  ],
  "request_skills": [
    "skill-name-if-additional-expertise-needed"
  ],
  "spawn_requests": [
    {
      "task": "Sub-task that needs delegation",
      "suggested_skills": ["skill-name"],
      "context": "Why this is needed",
      "priority": "high" | "medium" | "low"
    }
  ],
  "next_steps": [
    "Recommended follow-up actions"
  ]
}
```
</output-requirements>

## Execution Guidelines

1. **Follow the skill documentation precisely** - it contains project-specific patterns
2. **Use available tools** - Read, Edit, Write, Glob, Grep, Bash as needed
3. **Verify before completing** - check that your changes work
4. **Report blockers immediately** - don't guess if stuck
5. **Request skills if needed** - if you need expertise not in your skill-knowledge
```

---

## Compact Template (for simpler tasks)

```markdown
<skill>{SKILL CONTENT}</skill>

Task: {TASK}
Context: {CONTEXT}

Return JSON: { status, result, artifacts, issues }
```

---

## Multi-Skill Template

```markdown
You are an expert agent with knowledge across multiple domains.

<skills>

## api-route-generator
{content}

## component-generator
{content}

## linting-build-validator
{content}

</skills>

---

Task: {FULL FEATURE TASK}

Execute in order:
1. Create backend API using api-route-generator patterns
2. Create frontend component using component-generator patterns
3. Validate everything using linting-build-validator patterns

Return combined JSON with all artifacts.
```

---

## Refinement Template (for follow-up agents)

```markdown
You are a refinement agent. A previous agent completed work that needs improvement.

<previous-result>
{JSON output from previous agent}
</previous-result>

<additional-skill name="{REQUESTED_SKILL}">
{SKILL CONTENT}
</additional-skill>

<refinement-task>
Apply the additional skill knowledge to improve the previous result.
Focus on: {specific improvement areas}
</refinement-task>

Return updated JSON with refined artifacts.
```

---

## Validation Template (for chain validators)

```markdown
You are a validation agent. Review and validate work from previous agents.

<skill name="linting-build-validator">
{VALIDATOR SKILL CONTENT}
</skill>

<artifacts-to-validate>
{LIST OF FILES}
</artifacts-to-validate>

Validate all artifacts. Fix any issues found. Return:
{
  "status": "complete" if all valid, "needs_refinement" if issues remain,
  "validation_results": { ... },
  "fixes_applied": [ ... ],
  "remaining_issues": [ ... ]
}
```
