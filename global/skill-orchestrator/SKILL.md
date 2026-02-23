---
name: skill-orchestrator
description: Autonomous skill-aware orchestration system with dynamic skill discovery. Auto-detects available skills, matches tasks to skills, injects skill knowledge into sub-agents, and manages execution chains. Use when orchestrating complex tasks, spawning sub-agents, delegating work, building features, or any multi-step task that could benefit from specialized skill knowledge. This skill enables scale autonomy through self-discovering, skill-augmented agent spawning.
metadata:
  author: DDD
  version: 1.1.0
  category: orchestration
---

# Skill Orchestrator

## Purpose

Enable **fully autonomous, skill-aware agent orchestration** with:
- **Dynamic skill discovery** - no static configuration needed
- **Auto-matching** - tasks automatically match to relevant skills
- **Skill injection** - sub-agents receive expert knowledge via prompts
- **Execution chaining** - validation and refinement flows automatically
- **Self-maintaining** - new skills are instantly available

## Core Principle

**Skills are discovered, not configured.** Drop a SKILL.md file in a skills directory and it's immediately available to the orchestrator.

## Sub-Agent Routing Rules

**CRITICAL: These rules prevent the most common orchestration failure.**

When spawning skill-augmented agents:
- **ALWAYS use the Task tool** for agent invocation — DO NOT use bash commands, shell scripts, or CLI invocations to spawn sub-agents
- **Specify complete Task parameters**: `subagent_type` (use "general-purpose" for implementation tasks), `prompt` (with full skill content injected), `model` (optional)
- **Execute dependent steps sequentially** — if Step 2 requires output from Step 1, DO NOT spawn them in parallel. Wait for Step 1 to complete before spawning Step 2
- **Spawn independent steps in parallel** — if two agents create independent artifacts, spawn BOTH in a single message for true parallelism
- **Never invoke skills via bash** — DO NOT run `claude --skill X` or similar shell commands. Use the Skill tool for skill invocation within your own context, or inject skill content into Task tool prompts for sub-agents
- **Never let sub-agents spawn sub-agents via bash** — if a sub-agent needs additional help, it should return `needs_refinement` status and let the orchestrator handle the next spawn

---

## Hybrid Pattern: Native vs. Orchestrator

Claude Code supports **native `skills:` frontmatter** for custom agents. This orchestrator provides **complementary** capabilities. Use both together for maximum flexibility.

### When to Use What

| Scenario | Approach | Reason |
|----------|----------|--------|
| **Fixed specialist** (always same skills) | Native `skills:` frontmatter | Static binding is simpler, cleaner |
| **Dynamic task** ("build feature X") | skill-orchestrator | Auto-discovers and matches skills |
| **Repeatable pipeline** (deploy, full-feature) | Workflow YAML | Declarative, reproducible |
| **One-off complex task** | Ad-hoc orchestration | Adapts to unique requirements |
| **Known agent role** (db-admin, api-dev) | Native custom agent | Pre-defined expertise |
| **Unknown task scope** | skill-orchestrator | Explores and matches |

### Native `skills:` Frontmatter (Static)

For **custom agents in `.claude/agents/`** with fixed skill requirements:

```yaml
# .claude/agents/api-specialist.md
---
name: api-specialist
description: API development with all conventions loaded
tools: [Read, Write, Edit, Bash, Glob, Grep]
skills: [api-route-generator, linting-build-validator]
---
You are an API specialist. Create endpoints following loaded skill patterns.
```

**How it works:**
- Define agent once in `.claude/agents/`
- List skills statically in frontmatter
- Full skill content injected at subagent startup
- Invoke via `Task tool` with `subagent_type: api-specialist`

**Best for:** Roles that always need the same expertise (db-admin, ui-developer, etc.)

### skill-orchestrator (Dynamic)

For **Task tool** with dynamic skill matching:

```
User: "Build a user notifications feature"
    ↓
Orchestrator scans SKILL.md files
    ↓
Matches: api-route-generator, component-generator
    ↓
Loads skill content, injects into Task prompt
    ↓
Spawns agents with expertise baked in
    ↓
Chains validation automatically
```

**Best for:** Unknown tasks, feature requests, exploratory work

### What the Orchestrator Provides (Beyond Native)

| Capability | Native | skill-orchestrator |
|------------|--------|-------------------|
| Skill content → subagent | ✅ via `skills:` | ✅ via prompt injection |
| Works with custom agents | ✅ | ✅ |
| Works with standard Task tool | ❌ | ✅ **Dynamic** |
| Static skill binding | ✅ | N/A |
| Dynamic skill discovery | ❌ | ✅ **Novel** |
| Auto-matching from task | ❌ | ✅ **Novel** |
| Workflow pipelines | ❌ | ✅ **Novel** |
| Variable passing | ❌ | ✅ **Novel** |
| Conditions/stages | ❌ | ✅ **Novel** |
| Refinement loops | ❌ | ✅ **Novel** |

### Recommended Pattern

```
┌─────────────────────────────────────────────────────────────┐
│  1. FIXED ROLES → Native custom agents with skills:         │
│     .claude/agents/api-specialist.md                        │
│     .claude/agents/ui-specialist.md                         │
│     .claude/agents/infra-specialist.md                      │
└─────────────────────────────────────────────────────────────┘
                              +
┌─────────────────────────────────────────────────────────────┐
│  2. DYNAMIC TASKS → skill-orchestrator                      │
│     "Build feature X" → discover + match + execute          │
│     "Implement Y" → auto-select relevant skills             │
└─────────────────────────────────────────────────────────────┘
                              +
┌─────────────────────────────────────────────────────────────┐
│  3. REPEATABLE PIPELINES → Workflow YAML                    │
│     workflows/full-feature.yaml                             │
│     workflows/validate-and-deploy.yaml                      │
└─────────────────────────────────────────────────────────────┘
```

---

## PHASE 0: Dynamic Skill Discovery

**Run this FIRST on every orchestration request.**

### Step 0.1: Scan for Available Skills

Use Glob to find all SKILL.md files in known locations:

```
Skill Locations (scan in order):
1. Global user skills: ~/.claude/skills/**/SKILL.md
2. Project skills: ./.claude/skills/**/SKILL.md
3. Custom paths (from CLAUDE.md or settings)
4. Known plugin caches: ~/.claude/plugins/cache/*/skills/**/SKILL.md
```

For the user's setup, scan:
- `R:/CodingWork/my-claude-skills/global/**/SKILL.md`

### Step 0.2: Parse Each SKILL.md

Each SKILL.md has frontmatter between `---` delimiters:

```yaml
---
name: skill-name
description: Description with trigger keywords. Use when doing X, Y, Z...
---
```

Extract:
- `name` - the skill identifier
- `description` - contains trigger keywords after "Use when"
- `path` - full path to the file

### Step 0.3: Build Skill Index

Create an in-memory index:

```typescript
interface SkillIndex {
  [skillName: string]: {
    name: string;
    path: string;
    description: string;
    triggers: string[];  // extracted from description
  }
}
```

### Step 0.4: Extract Triggers

Parse triggers from the description field:
- Look for "Use when" followed by keywords
- Extract action words: "creating", "building", "checking", "validating", etc.
- Extract domain words: "api", "component", "ui", "deploy", "aws", etc.

Example:
```
description: "Create Next.js API routes... Use when creating api, new api route,
generate endpoint, build api, add endpoint..."

Extracted triggers: ["api", "route", "endpoint", "backend", "crud", "handler"]
```

---

## PHASE 1: Task Analysis & Skill Matching

### Step 1.1: Analyze the User's Task

Parse the task for:
- **Action intent**: create, build, fix, check, deploy, refactor
- **Domain signals**: api, component, ui, backend, database, aws
- **File types implied**: .ts routes, .tsx components, infrastructure

### Step 1.2: Match Against Skill Index

For each skill in the index:
```
score = 0
for each trigger in skill.triggers:
    if trigger appears in task (case-insensitive):
        score += 1
    if trigger appears in task context:
        score += 0.5

if score > threshold:
    matched_skills.append(skill)
```

### Step 1.3: Apply Smart Defaults

Always consider these chains:
- If ANY code-generating skill matched → include `linting-build-validator`
- If component/UI skill matched → include `design-system-auditor`
- If infrastructure skill matched → include relevant aws-* skills

### Step 1.4: Determine Execution Order

Build a simple DAG:
1. **Parallel group**: Skills that create independent artifacts
2. **Validation group**: Skills that check/validate (run after creation)
3. **Refinement**: Skills that improve based on validation feedback

Example for "build settings feature":
```
Parallel: [api-route-generator, component-generator]
    ↓
Sequential: [linting-build-validator]
    ↓
Sequential: [design-system-auditor] (if UI created)
    ↓
Refinement: (if issues found, loop back with combined skills)
```

---

## PHASE 2: Skill Content Loading

### Step 2.1: Load Matched Skills

For each matched skill:
1. Read the full SKILL.md file using Read tool
2. Extract content AFTER the frontmatter (after second `---`)
3. Store for injection

### Step 2.2: Prepare Injection Payloads

For each sub-agent to spawn:
```markdown
<skill-knowledge name="{skill.name}">
{full skill content, excluding frontmatter}
</skill-knowledge>
```

Multiple skills can be combined:
```markdown
<skill-knowledge name="api-route-generator">
{content}
</skill-knowledge>

<skill-knowledge name="linting-build-validator">
{content}
</skill-knowledge>
```

---

## PHASE 3: Orchestrated Execution

### Step 3.1: Spawn Parallel Agents

For skills in the parallel group, spawn ALL in a single message:

```
Task 1: {task for skill A} with skill A injected
Task 2: {task for skill B} with skill B injected
... (all in one message = true parallelism)
```

### Step 3.2: Collect Results

Each agent returns structured output:
```json
{
  "status": "complete" | "needs_refinement" | "blocked",
  "result": "what was accomplished",
  "artifacts": ["files created/modified"],
  "issues": ["problems encountered"],
  "request_skills": ["additional skills needed"],
  "spawn_requests": [{"task": "...", "skills": [...]}]
}
```

### Step 3.3: Handle Validation Chain

After parallel creation completes:
1. Spawn validation agent with `linting-build-validator` skill
2. Pass ALL artifacts from creation phase
3. If validation fails → proceed to refinement

### Step 3.4: Handle Refinement Loop

If any agent returns `needs_refinement`:

```
WHILE any result has status == "needs_refinement" AND attempts < 3:
    1. Identify requested_skills from the result
    2. Load those skill contents
    3. Spawn refinement agent with:
       - Original skill + requested skills combined
       - Previous result as context
       - Specific fix instructions
    4. Collect refined result
    5. Re-validate if needed
```

### Step 3.5: Synthesize Final Report

Combine all results into unified output:
- List all artifacts created
- Summarize what each agent accomplished
- Note any remaining issues
- Provide next steps if applicable

---

## Sub-Agent Prompt Template

Use this template when spawning skill-augmented agents:

**Invocation**: Use the Task tool with `subagent_type: "general-purpose"` and pass this template as the `prompt` parameter. DO NOT invoke sub-agents through bash, shell commands, or CLI tools.

```markdown
You are a specialized autonomous agent with expert knowledge.

<skill-knowledge name="{SKILL_NAME}">
{FULL SKILL CONTENT - everything after frontmatter}
</skill-knowledge>

{ADDITIONAL SKILLS IF COMBINED}

---

## Your Task

<task>
{SPECIFIC TASK DESCRIPTION}
</task>

<context>
- Project: {project type/name}
- Working directory: {path}
- Related files: {if applicable}
- Previous results: {if refinement}
</context>

<output-contract>
Return your response as JSON:
```json
{
  "status": "complete" | "needs_refinement" | "blocked",
  "result": "Clear description of what was accomplished",
  "artifacts": ["list", "of", "file", "paths"],
  "issues": ["any problems encountered"],
  "request_skills": ["skill-names-if-need-more-expertise"],
  "spawn_requests": [
    {
      "task": "sub-task description",
      "suggested_skills": ["skill-names"]
    }
  ]
}
```
</output-contract>

Execute the task using the skill knowledge provided. Follow all patterns exactly.
```

---

## Automatic Skill Adoption

When a new skill is added:

1. **No action needed** - next orchestration will discover it
2. Skill's `description` field provides triggers
3. Skill's content provides the expertise

To add a new skill:
```
1. Create folder: skills/global/my-new-skill/
2. Create SKILL.md with frontmatter:
   ---
   name: my-new-skill
   description: Does X. Use when doing A, B, C...
   ---
   # Skill content here...
3. Done. It's now discoverable.
```

---

## Quick Reference: Orchestration Flow

```
┌─────────────────────────────────────────────────────────────┐
│  USER REQUEST                                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 0: DISCOVER                                          │
│  Glob for SKILL.md files → Parse frontmatter → Build index  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: MATCH                                             │
│  Analyze task → Match triggers → Determine execution order  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: LOAD                                              │
│  Read matched SKILL.md files → Prepare injection payloads   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: EXECUTE                                           │
│  Spawn parallel agents → Collect results → Chain validation │
│       ↓                                                     │
│  If needs_refinement → Loop with additional skills          │
│       ↓                                                     │
│  Synthesize final report                                    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  COMPLETE: Artifacts + Summary + Next Steps                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Example: Full Orchestration

**User**: "Build a user notifications feature"

**Phase 0 - Discovery**:
```
Scanning R:/CodingWork/my-claude-skills/global/**/SKILL.md...
Found 11 skills: api-route-generator, component-generator, design-system-auditor,
linting-build-validator, changelog-generator, secure-deployment-manager,
aws-infrastructure-manager, aws-cdk-development, aws-serverless-eda,
ralph-wiggum-prompts, skill-orchestrator
```

**Phase 1 - Matching**:
```
Task: "Build a user notifications feature"
Signals: "build" (creation), "feature" (full-stack), "notifications" (domain)

Matches:
- api-route-generator (triggers: build, api, endpoint) → score: 2
- component-generator (triggers: build, component, ui) → score: 2
- linting-build-validator (auto-include for code generation)
- design-system-auditor (auto-include for UI)

Execution plan:
  Parallel: [api-route-generator, component-generator]
  Chain: [linting-build-validator, design-system-auditor]
```

**Phase 2 - Loading**:
```
Reading: global/api-route-generator/SKILL.md (691 lines)
Reading: global/component-generator/SKILL.md (450 lines)
Reading: global/linting-build-validator/SKILL.md (482 lines)
Reading: global/design-system-auditor/SKILL.md (380 lines)
```

**Phase 3 - Execution**:
```
Spawning Agent A (api-route-generator): Create /api/notifications route
Spawning Agent B (component-generator): Create NotificationCenter component
[PARALLEL - both in same message]

Results collected:
- Agent A: complete, artifacts: [src/app/api/notifications/route.ts]
- Agent B: complete, artifacts: [src/components/notifications/notification-center.tsx]

Spawning Agent C (linting-build-validator): Validate both files
Result: complete, no issues

Spawning Agent D (design-system-auditor): Audit component
Result: complete, 95% compliance

Final synthesis: Feature complete, 2 files created, all checks passed.
```

---

## Integration with CLAUDE.md

Add to project CLAUDE.md for automatic orchestration:

```markdown
## Skill Orchestration

Before responding to implementation requests:
1. Invoke skill-orchestrator pattern
2. Discover available skills
3. Match task to skills
4. Execute with skill-augmented agents
5. Validate and refine automatically

Skill locations:
- Global: R:/CodingWork/my-claude-skills/global/
- Project: ./.claude/skills/
```

---

## Best Practices

1. **Trust discovery** - don't manually configure skills
2. **Write good descriptions** - triggers come from descriptions
3. **Use "Use when" pattern** - makes trigger extraction reliable
4. **Keep skills focused** - one skill = one expertise domain
5. **Include examples** - skills with examples produce better results
6. **Chain validators** - always validate generated code
7. **Handle refinement** - expect and plan for iteration

---

## WORKFLOW EXECUTION

### Overview

Workflows are declarative YAML pipelines that define complex, multi-stage automation. They take priority over ad-hoc skill matching when a user request matches a workflow trigger.

### Execution Priority

```
User Request
    ↓
1. Check workflows/*.yaml for trigger match
   → If match: Execute workflow pipeline
    ↓
2. If no workflow: Dynamic skill discovery
   → Match skills, build DAG, execute
    ↓
3. If no skills: Normal response
```

### Workflow Discovery

Scan for workflow files:
```
Locations:
1. ~/.claude/workflows/*.yaml
2. ./.claude/workflows/*.yaml
3. {skills-repo}/workflows/*.yaml
```

### Trigger Matching

Workflows define trigger patterns:
```yaml
trigger:
  - "build feature *"      # Matches "build feature notifications"
  - "deploy to *"          # Matches "deploy to staging"
```

When user request matches, extract variables:
- "build feature notifications" → `$feature = "notifications"`

### Workflow Execution Steps

**Step 1: Parse Workflow**
```yaml
name: Full Feature Pipeline
stages:
  - name: Backend
    skills: [api-route-generator]
    parallel: true
  - name: Frontend
    skills: [component-generator]
    parallel: true
  - name: Validate
    depends_on: [Backend, Frontend]
    skills: [linting-build-validator]
```

**Step 2: Build Execution DAG**
```
Group 1 (parallel): [Backend, Frontend]
Group 2 (sequential): [Validate]  # depends on Group 1
```

**Step 3: Execute Groups**
For each group:
1. Load skills for all stages in group
2. Interpolate variables in task descriptions
3. Evaluate conditions (skip if false)
4. Spawn agents (parallel if group is parallel)
5. Collect results
6. Pass results to dependent stages

**Step 4: Handle Conditions**
```yaml
- name: Refinement
  condition: "$Validate.status == 'needs_refinement'"
```
Only execute if condition evaluates to true.

**Step 5: Synthesize Results**
Combine all stage outputs into final report.

### Variable System

**Built-in Variables:**
- `$feature` - extracted from trigger match
- `$project` - current project name
- `$timestamp` - current time
- `$StageName.field` - reference stage output

**Interpolation:**
```yaml
task: "Create API for {$feature}"
# Becomes: "Create API for notifications"
```

### Available Workflows

Located in `workflows/` directory:

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `full-feature.yaml` | "build feature *" | Complete feature with API, UI, validation |
| `validate-and-deploy.yaml` | "deploy to *" | Pre-flight checks + deployment |
| `quick-validate.yaml` | "check my code" | Fast pre-commit validation |

### Creating New Workflows

1. Create `.yaml` file in workflows directory
2. Define name, trigger, and stages
3. Done - automatically discoverable

Example minimal workflow:
```yaml
name: My Workflow
trigger: "do something *"
stages:
  - name: Step1
    skills: [some-skill]
    task: "Do the thing for {$1}"
```

### Workflow vs Ad-hoc Orchestration

| Aspect | Workflow | Ad-hoc |
|--------|----------|--------|
| Definition | Declarative YAML | Dynamic discovery |
| Reproducibility | Exact same every time | May vary |
| Customization | Pre-defined stages | Adapts to task |
| Best for | Repeatable pipelines | One-off tasks |

Use **workflows** for standard processes you run repeatedly.
Use **ad-hoc** for unique or exploratory tasks.

---

## Complete System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    SKILL ORCHESTRATOR                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  WORKFLOWS  │    │   SKILLS    │    │   AGENTS    │         │
│  │  (*.yaml)   │    │ (SKILL.md)  │    │   (Task)    │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ORCHESTRATION ENGINE                        │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  1. Discover workflows & skills                         │   │
│  │  2. Match request to triggers                           │   │
│  │  3. Build execution plan (DAG)                          │   │
│  │  4. Load skill content for matched skills               │   │
│  │  5. Spawn skill-augmented agents                        │   │
│  │  6. Collect & chain results                             │   │
│  │  7. Handle refinement loops                             │   │
│  │  8. Synthesize final output                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│                    ┌─────────────────┐                         │
│                    │  FINAL RESULT   │                         │
│                    │  + Artifacts    │                         │
│                    │  + Summary      │                         │
│                    │  + Next Steps   │                         │
│                    └─────────────────┘                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

This system provides:
- **Zero-config skill adoption** - drop SKILL.md, immediately usable
- **Declarative workflows** - repeatable pipelines via YAML
- **Dynamic orchestration** - intelligent ad-hoc task handling
- **Skill injection** - sub-agents operate with expert knowledge
- **Automatic chaining** - validation and refinement flows naturally
- **Self-improving** - refinement loops until success

## Troubleshooting

### Sub-agent fails to use injected skill

**Symptom**: Agent receives skill content in prompt but ignores it, uses generic approach instead.

**Cause**: Skill content buried too deep in the prompt, or prompt is too long and skill instructions get lost in context.

**Fix**: Move the `<skill-knowledge>` block earlier in the prompt. Keep the total prompt under 4000 words — if longer, extract reference material to files the agent can read on demand.

### Skill not discovered during scan

**Symptom**: `Glob("**/SKILL.md")` doesn't find a skill you know exists.

**Cause**: File not named exactly `SKILL.md` (case-sensitive), or file is a loose `.md` not inside a skill folder.

**Fix**: Verify the skill follows `{skill-name}/SKILL.md` folder structure. Check for common naming issues: `skill.md`, `SKILL.MD`, or `Skill.md`.

### Sub-agent spawned via bash instead of Task tool

**Symptom**: Agent tries to run `claude --skill X` or similar shell command instead of using Task tool.

**Cause**: Prompt doesn't explicitly prohibit bash invocation, or agent defaults to CLI patterns.

**Fix**: Add explicit routing rules to the prompt: "Use the Task tool for agent invocation — DO NOT use bash commands." See the Sub-Agent Routing Rules section.

### Dependent steps executed in parallel

**Symptom**: Step 2 starts before Step 1 completes, produces errors or uses stale data.

**Cause**: Steps incorrectly identified as independent, or parallelism instructions unclear.

**Fix**: Explicitly mark dependencies: "Step 2 depends on Step 1 output. DO NOT parallelize." Only parallelize steps that produce independent artifacts with no shared state.

### needs_refinement loop doesn't converge

**Symptom**: Agent keeps returning needs_refinement status, skill enters infinite refinement cycle.

**Cause**: Verification criteria too strict, or the agent can't meet the quality bar with available context.

**Fix**: Set a max refinement count (default: 2). After max attempts, return the best result with a note about what didn't pass. Let the orchestrator decide whether to accept or escalate.
