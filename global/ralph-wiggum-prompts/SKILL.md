---
name: ralph-wiggum-prompts
description: Use when preparing work for Ralph Wiggum autonomous loop, structuring tasks for iterative execution, or when user mentions "ralph", "wiggum", or "loop prompt". Transforms plans into prompts with completion criteria and <promise> markers.
metadata:
  author: DDD
  version: 1.0.0
  category: automation
---

# Ralph Wiggum Prompt Engineering

Transform user plans, tasks, features, and requests into prompts optimized for the Ralph Wiggum autonomous loop.

**Announce:** "Using ralph-wiggum-prompts skill to structure this for autonomous execution."

## How the Loop Works

The loop repeatedly feeds Claude a prompt until completion:
```bash
/ralph-loop:ralph-loop "$(cat docs/ralph-prompts/task.md)" --completion-promise "COMPLETE" --max-iterations 10
```

Success depends on prompt quality: self-contained, incremental phases, self-correcting, explicit completion signal.

## Transformation Process

1. **Analyze input** - Identify type (plan, feature, bug, refactor, vague)
2. **Clarify if needed** - "What does done look like?" / "What commands verify success?"
3. **Select template** - Match task type to template
4. **Structure prompt** - Context, requirements, phases, verification, completion
5. **Write to file** - Save to `docs/ralph-prompts/<task-name>.md`
6. **Output command** - Ready-to-run `/ralph-loop` invocation

## Template Selection

| Task Type | Template | Key Pattern |
|-----------|----------|-------------|
| New feature | [feature-template.md](references/feature-template.md) | Phased implementation |
| TDD work | [tdd-template.md](references/tdd-template.md) | Red-Green-Refactor cycles |
| Bug fix | [bugfix-template.md](references/bugfix-template.md) | Reproduce-Fix-Verify |
| Refactoring | [refactor-template.md](references/refactor-template.md) | Safety-first incremental |

## Required Prompt Sections

Every Ralph prompt needs: Context, Requirements (checkboxes), Implementation Phases (atomic steps), Completion Criteria, Verification Commands, Self-Correction Protocol, and the `<promise>COMPLETE</promise>` marker.

See [writing-rules.md](references/writing-rules.md) for detailed guidelines and [anti-patterns.md](references/anti-patterns.md) for what to avoid.

## Output Format

1. Create `docs/ralph-prompts/` directory
2. Write structured prompt to `docs/ralph-prompts/<task-name>.md`
3. Output ready command:
   ```bash
   /ralph-loop:ralph-loop "$(cat docs/ralph-prompts/<task-name>.md)" --max-iterations <N> --completion-promise "<text>"
   ```
4. Summarize: file path, recommended iterations, watch points
