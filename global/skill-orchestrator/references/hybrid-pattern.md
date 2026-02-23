# Hybrid Pattern: Native Agents + skill-orchestrator

## Overview

Claude Code provides **two complementary mechanisms** for skill-augmented agents:

1. **Native `skills:` frontmatter** - Static binding for custom agents
2. **skill-orchestrator** - Dynamic discovery, matching, and workflows

This document explains when to use each and how they work together.

---

## The Key Distinction

### Native `skills:` Frontmatter

```
Define custom agent → List skills statically → Agent always has those skills
```

**How it works:**
- Create agent file in `.claude/agents/`
- List skills in frontmatter `skills:` field
- Full skill content injected at subagent startup
- Invoke via Task tool with custom `subagent_type`

**Example:**
```yaml
# .claude/agents/api-specialist.md
---
name: api-specialist
skills: [api-route-generator, linting-build-validator]
---
```

When you spawn `subagent_type: api-specialist`, the agent automatically has the full content of both skills loaded into context.

### skill-orchestrator (This System)

```
User says "build feature X" → Discover all skills → Match triggers →
Inject matched skills into Task prompt → Execute → Chain validation →
Refine if needed → Synthesize
```

**How it works:**
- Uses standard Task tool (general-purpose, Explore, etc.)
- Dynamically discovers relevant skills by scanning SKILL.md files
- Matches based on task description keywords
- Injects only what's needed
- Supports workflows, variables, conditions

---

## Comparison Table

| Aspect | Native `skills:` | skill-orchestrator |
|--------|------------------|-------------------|
| **Binding** | Static (defined at agent creation) | Dynamic (discovered at runtime) |
| **Discovery** | Manual listing | Automatic via glob |
| **Custom agents** | Required | Optional |
| **Standard Task tool** | ❌ No skill injection | ✅ Full skill injection |
| **Workflows** | ❌ Not supported | ✅ YAML pipelines |
| **Variable passing** | ❌ | ✅ $Stage.field |
| **Conditions** | ❌ | ✅ |
| **Refinement loops** | ❌ | ✅ |
| **Best for** | Fixed roles | Unknown tasks |

---

## When to Use Each

### Use Native `skills:` When:

1. **You have a well-defined specialist role**
   - "API developer" always needs api-route-generator + linting-build-validator
   - "UI developer" always needs component-generator + design-system-auditor

2. **The skill set is predictable**
   - Database work → always needs db patterns
   - Infrastructure → always needs AWS skills

3. **You want simplicity**
   - One-time setup, always works
   - No runtime discovery overhead

### Use skill-orchestrator When:

1. **The task is dynamic/unknown**
   - "Build feature X" - need to discover what's relevant
   - "Implement Y" - could be API, UI, or both

2. **You need workflow pipelines**
   - Multi-stage execution
   - Parallel + sequential phases
   - Conditional branches

3. **You need refinement loops**
   - Validation fails → retry with more skills
   - Design audit fails → fix and re-check

4. **You want zero configuration**
   - Drop SKILL.md, immediately available
   - No agent file needed

---

## Hybrid Pattern in Practice

### Recommended Setup

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 1: Native Custom Agents (Fixed Roles)                │
│                                                             │
│  .claude/agents/                                            │
│  ├── api-specialist.md     [api-route-generator, lint]      │
│  ├── ui-specialist.md      [component-generator, design]    │
│  └── infra-specialist.md   [aws-*, deployment]              │
│                                                             │
│  → Use when you KNOW the task type                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  LAYER 2: skill-orchestrator (Dynamic Tasks)                │
│                                                             │
│  "Build feature notifications"                              │
│      → Discovers: api-route-generator, component-generator  │
│      → Injects both into Task prompts                       │
│      → Chains validation automatically                      │
│                                                             │
│  → Use when task scope is UNKNOWN                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  LAYER 3: Workflows (Repeatable Pipelines)                  │
│                                                             │
│  workflows/full-feature.yaml                                │
│  workflows/validate-and-deploy.yaml                         │
│                                                             │
│  → Use for STANDARDIZED processes                           │
└─────────────────────────────────────────────────────────────┘
```

### Decision Flowchart

```
┌─────────────────────────────────────────┐
│  User requests task                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Is this a known, repeatable pipeline?   │
│  (deploy, full-feature, validate)        │
└────────────────┬────────────────────────┘
                 │
        ┌────────┴────────┐
        │ YES             │ NO
        ▼                 ▼
┌───────────────┐  ┌─────────────────────────────┐
│ Use Workflow  │  │ Is task type well-defined?   │
│ YAML pipeline │  │ (pure API, pure UI, etc.)    │
└───────────────┘  └────────────┬────────────────┘
                                │
                       ┌────────┴────────┐
                       │ YES             │ NO
                       ▼                 ▼
               ┌───────────────┐  ┌─────────────────┐
               │ Use Native    │  │ Use skill-      │
               │ Custom Agent  │  │ orchestrator    │
               │ (api-spec,    │  │ for dynamic     │
               │  ui-spec)     │  │ discovery       │
               └───────────────┘  └─────────────────┘
```

---

## Example: Same Task, Different Approaches

### Task: "Create a notifications API endpoint"

**Approach 1: Native Custom Agent**
```
Task tool → subagent_type: api-specialist
"Create notifications API endpoint"

Agent already has api-route-generator loaded → executes immediately
```

**Approach 2: skill-orchestrator**
```
Task tool → general-purpose with orchestrator skill
"Create notifications API endpoint"

Orchestrator:
1. Scans SKILL.md files
2. Matches: api-route-generator (triggers: api, endpoint, create)
3. Auto-includes: linting-build-validator
4. Injects skills into prompt
5. Spawns agent
6. Chains validation
```

**Both work.** Native is simpler for known task types. Orchestrator is more flexible for unknown tasks.

---

## What skill-orchestrator Uniquely Provides

These capabilities have **no native equivalent**:

### 1. Workflow YAML Pipelines
```yaml
stages:
  - name: Backend
    parallel: true
  - name: Frontend
    parallel: true
  - name: Validate
    depends_on: [Backend, Frontend]
```

### 2. Variable Passing
```yaml
input: $Backend.artifacts
task: "Validate files: {input}"
```

### 3. Conditional Execution
```yaml
condition: "$Validate.status == 'needs_refinement'"
```

### 4. Refinement Loops
```
If needs_refinement → load additional skills → retry → re-validate
```

### 5. Dynamic Discovery
```
Drop SKILL.md anywhere in known paths → instantly available
No configuration needed
```

---

## Implementation Notes

### Creating a Native Custom Agent

1. Create file: `.claude/agents/my-specialist.md`
2. Add frontmatter with `skills:` field
3. Write agent instructions in body
4. Invoke with `subagent_type: my-specialist`

### Using skill-orchestrator

1. Ensure skill-orchestrator SKILL.md is discoverable
2. Claude auto-loads it based on task keywords
3. Orchestrator handles discovery, matching, execution
4. No additional setup needed

### Best of Both

```yaml
# .claude/agents/feature-builder.md
---
name: feature-builder
skills: [skill-orchestrator]  # Load orchestrator as a skill!
---
You are a feature builder. Use the skill-orchestrator patterns to:
1. Discover relevant skills for the requested feature
2. Execute with proper chaining
3. Validate and refine
```

This gives you a custom agent that **uses orchestrator patterns** - combining both approaches.
