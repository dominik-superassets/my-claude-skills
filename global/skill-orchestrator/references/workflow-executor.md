# Workflow Executor

## Overview

The workflow executor interprets YAML workflow definitions and orchestrates their execution through skill-augmented agents.

## Execution Flow

```
User Request: "build feature notifications"
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: WORKFLOW DISCOVERY                                 │
│  Glob for *.yaml in workflow directories                    │
│  Parse each workflow's trigger patterns                     │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 2: TRIGGER MATCHING                                   │
│  Match user request against all workflow triggers           │
│  Select best matching workflow                              │
│  Extract variables from trigger pattern                     │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 3: WORKFLOW PARSING                                   │
│  Load matched workflow YAML                                 │
│  Parse stages, dependencies, conditions                     │
│  Build execution DAG                                        │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 4: STAGE EXECUTION                                    │
│  For each stage (respecting dependencies):                  │
│    - Load required skills                                   │
│    - Interpolate variables                                  │
│    - Evaluate conditions                                    │
│    - Spawn skill-augmented agent                            │
│    - Collect results                                        │
│    - Handle retries if needed                               │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 5: SYNTHESIS                                          │
│  Combine all stage results                                  │
│  Generate final report                                      │
│  Return to user                                             │
└─────────────────────────────────────────────────────────────┘
```

## Step-by-Step Execution Guide

### Step 1: Discover Available Workflows

```
Workflow locations to scan:
1. ~/.claude/workflows/*.yaml
2. ./.claude/workflows/*.yaml
3. {skills-repo}/workflows/*.yaml

For each .yaml file:
  - Read the file
  - Extract 'name' and 'trigger' fields
  - Store in workflow index
```

### Step 2: Match User Request to Workflow

```python
def match_workflow(request: str, workflows: list) -> Workflow:
    for workflow in workflows:
        for trigger in workflow.triggers:
            if matches(request, trigger):
                variables = extract_variables(request, trigger)
                return (workflow, variables)
    return None

def matches(request: str, pattern: str) -> bool:
    # Convert pattern to regex
    # "build feature *" → "^build feature (.+)$"
    regex = pattern.replace("*", "(.+)")
    return re.match(regex, request, re.IGNORECASE)

def extract_variables(request: str, pattern: str) -> dict:
    # "build feature notifications" with "build feature *"
    # → { "feature": "notifications" }
    ...
```

### Step 3: Build Execution DAG

```python
def build_dag(stages: list) -> ExecutionPlan:
    # Group stages by dependencies
    groups = []

    # Find stages with no dependencies (can run first)
    ready = [s for s in stages if not s.depends_on]

    # Find parallel groups
    parallel_group = [s for s in ready if s.parallel]
    if parallel_group:
        groups.append({"parallel": True, "stages": parallel_group})

    sequential = [s for s in ready if not s.parallel]
    for s in sequential:
        groups.append({"parallel": False, "stages": [s]})

    # Continue building DAG for dependent stages...
    # (recursive dependency resolution)

    return groups
```

### Step 4: Execute Stages

For each stage group:

```python
async def execute_stage(stage, context, variables):
    # 1. Check condition
    if stage.condition:
        if not evaluate_condition(stage.condition, context):
            return {"status": "skipped", "reason": "condition not met"}

    # 2. Load skills
    skills_content = []
    for skill_name in stage.skills:
        skill = discover_skill(skill_name)
        content = read_skill_content(skill.path)
        skills_content.append((skill_name, content))

    # 3. Interpolate variables in task
    task = interpolate(stage.task, variables, context)

    # 4. Compose agent prompt
    prompt = compose_prompt(skills_content, task, stage.context)

    # 5. Spawn agent
    result = await spawn_agent(
        prompt=prompt,
        agent_type=stage.agent or "general-purpose",
        model=stage.model or "sonnet"
    )

    # 6. Handle retries
    attempts = 0
    while result.status == "needs_refinement" and attempts < stage.retry:
        result = await retry_with_feedback(stage, result)
        attempts += 1

    return result
```

### Step 5: Variable Interpolation

```python
def interpolate(template: str, variables: dict, context: dict) -> str:
    result = template

    # Replace {$varname} patterns
    for key, value in variables.items():
        result = result.replace(f"{{$feature}}", value)

    # Replace {$StageName.field} patterns
    for stage_name, stage_result in context.items():
        for field, value in stage_result.items():
            pattern = f"{{${stage_name}.{field}}}"
            result = result.replace(pattern, str(value))

    return result
```

### Step 6: Condition Evaluation

```python
def evaluate_condition(condition: str, context: dict) -> bool:
    # Simple conditions
    # "$Backend.status == 'complete'" → context["Backend"]["status"] == "complete"

    # Parse condition
    match = re.match(r'\$(\w+)\.(\w+)\s*(==|!=)\s*[\'"](.+)[\'"]', condition)
    if match:
        stage, field, op, value = match.groups()
        actual = context.get(stage, {}).get(field)
        if op == "==":
            return actual == value
        elif op == "!=":
            return actual != value

    return True  # Default to true if can't parse
```

## Parallel Execution

When executing parallel stages, spawn ALL in a single message:

```python
async def execute_parallel_group(stages, context, variables):
    # Compose all prompts
    tasks = []
    for stage in stages:
        skills_content = load_skills(stage.skills)
        task = interpolate(stage.task, variables, context)
        prompt = compose_prompt(skills_content, task, stage.context)
        tasks.append((stage.name, prompt, stage.agent, stage.model))

    # Spawn all agents in single message (true parallelism)
    results = await spawn_agents_parallel(tasks)

    # Map results back to stage names
    return {name: result for name, result in zip([t[0] for t in tasks], results)}
```

## Error Handling

```python
async def execute_with_error_handling(stage, context, variables):
    try:
        result = await execute_stage(stage, context, variables)

        if result.status == "blocked":
            if stage.on_failure == "stop":
                raise WorkflowStopException(result)
            elif stage.on_failure == "continue":
                return result
            elif stage.on_failure == "skip_dependents":
                mark_dependents_skipped(stage.name)
                return result

        return result

    except Exception as e:
        if stage.fallback:
            return await execute_fallback(stage.fallback, context, variables)
        raise
```

## Full Execution Example

```
Workflow: "Full Feature Pipeline"
Trigger: "build feature notifications"
Variables: { feature: "notifications" }

Execution:
├── Group 1 (parallel):
│   ├── Backend (api-route-generator)
│   │   → Task: "Create API endpoint for notifications"
│   │   → Result: { status: "complete", artifacts: [...] }
│   │
│   └── Frontend (component-generator)
│       → Task: "Create UI component for notifications"
│       → Result: { status: "complete", artifacts: [...] }
│
├── Group 2 (depends on Group 1):
│   └── LintValidation (linting-build-validator)
│       → Input: [Backend.artifacts, Frontend.artifacts]
│       → Result: { status: "complete", issues: [] }
│
├── Group 3 (depends on Group 2):
│   └── DesignAudit (design-system-auditor)
│       → Condition: Frontend.status == "complete" ✓
│       → Result: { status: "complete", compliance: "95%" }
│
├── Group 4 (conditional):
│   └── Refinement
│       → Condition: needs_refinement? ✗ (skipped)
│
└── Group 5:
    └── FinalReport
        → Synthesize all results
        → Return to user

Total: 5 stages executed, 1 skipped
Artifacts: 2 files created
Status: COMPLETE
```

## Integration with Skill Orchestrator

The workflow executor is invoked automatically when:

1. User request matches a workflow trigger
2. OR user explicitly requests: "run workflow X"
3. OR workflow is referenced in CLAUDE.md automation

Priority order:
1. Check for matching workflow first
2. If no workflow matches, fall back to dynamic skill discovery
3. If skill discovery finds matches, execute with orchestrator
4. If nothing matches, respond normally

This creates a seamless experience where complex pipelines are triggered naturally.
