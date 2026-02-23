# Workflow Composition Schema

## Overview

Workflows are declarative YAML files that define multi-stage execution pipelines. They enable complex, repeatable automation with skill-augmented agents.

## File Locations

```
Workflow Discovery Paths:
1. ~/.claude/workflows/*.yaml           # Global user workflows
2. ./.claude/workflows/*.yaml           # Project workflows
3. {skills-repo}/workflows/*.yaml       # Shared workflows
```

## Schema Definition

```yaml
# Required fields
name: string                    # Human-readable workflow name
trigger: string | string[]      # Pattern(s) that activate this workflow

# Optional metadata
description: string             # What this workflow does
version: string                 # Semantic version
author: string                  # Creator

# Execution definition
stages: Stage[]                 # Ordered list of execution stages

# Optional settings
settings:
  max_retries: number           # Default retry count (default: 2)
  timeout_minutes: number       # Max execution time (default: 30)
  fail_fast: boolean            # Stop on first failure (default: true)
  notify_on_complete: boolean   # Alert user when done (default: true)
```

## Stage Schema

```yaml
stages:
  - name: string                # Stage identifier (used for $references)

    # What to execute
    skills: string[]            # Skills to inject into agent
    agent: string               # Agent type (default: general-purpose)
    model: string               # Model to use (default: sonnet)

    # Task definition
    task: string                # Task description (supports $variables)
    context: string             # Additional context

    # Execution control
    parallel: boolean           # Can run with other parallel:true stages
    depends_on: string[]        # Stage names that must complete first
    condition: string           # Only run if condition is true

    # Input/Output
    input: any                  # Data from previous stages ($StageName.field)
    output: string              # Variable name for this stage's output

    # Error handling
    retry: number               # Retry count for this stage
    on_failure: string          # Action: "stop" | "continue" | "skip_dependents"
    fallback: string            # Alternative stage to run on failure
```

## Variable System

### Stage Output References

Each stage produces output accessible via `$StageName`:

```yaml
stages:
  - name: Backend
    skills: [api-route-generator]
    task: "Create API for {feature}"
    # Output: { status, result, artifacts, issues }

  - name: Validate
    skills: [linting-build-validator]
    input: $Backend.artifacts    # Reference previous stage output
    task: "Validate files: {input}"
```

### Built-in Variables

```yaml
$feature          # Extracted from trigger match
$project          # Current project name
$timestamp        # Current ISO timestamp
$user             # Current user info
$previous         # Previous stage output (shorthand)
$all_artifacts    # Combined artifacts from all prior stages
```

### Variable Interpolation

```yaml
task: "Create {$feature} API endpoint"
# With trigger match "build feature notifications"
# Becomes: "Create notifications API endpoint"
```

## Parallel Execution

Stages with `parallel: true` run simultaneously:

```yaml
stages:
  - name: Backend
    parallel: true              # ─┬─ These run together
    skills: [api-route-generator]

  - name: Frontend
    parallel: true              # ─┘
    skills: [component-generator]

  - name: Validate
    depends_on: [Backend, Frontend]  # Waits for both
    skills: [linting-build-validator]
```

## Conditional Execution

```yaml
stages:
  - name: CreateAPI
    skills: [api-route-generator]

  - name: DeployStaging
    condition: "$CreateAPI.status == 'complete'"
    skills: [secure-deployment-manager]

  - name: NotifyFailure
    condition: "$CreateAPI.status == 'blocked'"
    task: "Report failure to user"
```

## Error Handling

```yaml
stages:
  - name: RiskyOperation
    skills: [some-skill]
    retry: 3
    on_failure: continue        # Don't stop pipeline

  - name: Fallback
    condition: "$RiskyOperation.status != 'complete'"
    task: "Alternative approach"
```

## Trigger Patterns

### Simple Triggers

```yaml
trigger: "build feature *"      # Matches "build feature X"
trigger: "deploy to *"          # Matches "deploy to staging"
```

### Multiple Triggers

```yaml
trigger:
  - "build feature *"
  - "create feature *"
  - "implement *"
```

### Regex Triggers

```yaml
trigger: "/^(build|create|implement)\s+(\w+)\s+feature$/i"
```

### Trigger Captures

```yaml
trigger: "build {type} for {target}"
# "build api for users" → $type = "api", $target = "users"
```

## Complete Example

```yaml
name: Full Feature Pipeline
description: Create complete feature with backend, frontend, and validation
version: "1.0.0"
trigger:
  - "build feature *"
  - "create * feature"
  - "implement *"

settings:
  max_retries: 2
  fail_fast: false

stages:
  # Stage 1 & 2: Parallel creation
  - name: Backend
    skills: [api-route-generator]
    parallel: true
    task: "Create API endpoint for {$feature}"
    context: "Follow all auth and permission patterns"

  - name: Frontend
    skills: [component-generator, design-system-auditor]
    parallel: true
    task: "Create UI component for {$feature}"
    context: "Include dark mode, hover states, proper spacing"

  # Stage 3: Validation (after creation completes)
  - name: Validate
    skills: [linting-build-validator]
    depends_on: [Backend, Frontend]
    input:
      files:
        - $Backend.artifacts
        - $Frontend.artifacts
    task: "Validate all created files for linting and type errors"
    retry: 2

  # Stage 4: Design audit (after validation)
  - name: DesignAudit
    skills: [design-system-auditor]
    depends_on: [Validate]
    condition: "$Frontend.artifacts.length > 0"
    input: $Frontend.artifacts
    task: "Audit UI components for design system compliance"

  # Stage 5: Refinement (if needed)
  - name: Refinement
    skills: [component-generator, linting-build-validator]
    condition: "$DesignAudit.status == 'needs_refinement'"
    input:
      issues: $DesignAudit.issues
      files: $Frontend.artifacts
    task: "Fix design system violations: {$DesignAudit.issues}"

  # Stage 6: Final report
  - name: Report
    depends_on: [Validate, DesignAudit]
    task: |
      Synthesize final report:
      - Backend: $Backend.result
      - Frontend: $Frontend.result
      - Validation: $Validate.result
      - Design: $DesignAudit.result
      - All artifacts: $all_artifacts
```

## Workflow Output

Each workflow execution produces:

```json
{
  "workflow": "Full Feature Pipeline",
  "trigger_match": "build feature notifications",
  "variables": {
    "feature": "notifications"
  },
  "stages": {
    "Backend": { "status": "complete", "artifacts": [...] },
    "Frontend": { "status": "complete", "artifacts": [...] },
    "Validate": { "status": "complete", "issues": [] },
    "DesignAudit": { "status": "complete", "compliance": "95%" },
    "Refinement": { "status": "skipped", "reason": "condition not met" },
    "Report": { "status": "complete" }
  },
  "overall_status": "complete",
  "total_artifacts": [...],
  "execution_time": "45s"
}
```
