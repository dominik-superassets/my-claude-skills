---
name: prompt-architect
description: >
  On-demand prompt structurer that transforms vague requests into well-architected
  prompts with execution strategy. Use when user says "refine this prompt", "prompt check",
  "structure this", "plan this out", "architect this", or indicates they want strategic
  thinking before execution. For complex multi-step tasks, ambiguous goals, or high-stakes
  deliverables. Do NOT trigger on simple factual questions, quick fixes, or when the user
  is clearly ready to execute.
metadata:
  author: DDD
  version: 1.0.0
  category: workflow
---

# Prompt Architect

Transforms rough intent into structured, high-quality prompts. Inspired by how production AI agents decompose complex tasks into executable contracts.

This is NOT a universal interceptor. It activates when explicitly requested or when task complexity warrants strategic thinking before execution.

## Core Philosophy

A well-structured prompt is a **behavioral contract** with six parts. Most prompts only cover 1-2. This skill ensures all six are addressed when they matter:

1. **Identity** → What role/expertise is needed
2. **Routing** → Which approach fits this specific variant
3. **Choreography** → Step-by-step execution order with checkpoints
4. **Verification** → How to know the output is correct
5. **Recovery** → What to do when something goes wrong
6. **Boundaries** → What to explicitly avoid

Not every prompt needs all six. A quick question needs zero. A 20-slide deck needs all of them. Calibrate depth to complexity.

## Execution Flow

### Step 1: Assess Complexity

| Complexity | Characteristics | Depth |
|-----------|----------------|-------|
| **Light** | Single output, clear format, no ambiguity | Quick Reframe — just tighten the ask |
| **Medium** | Multi-step, some ambiguity, 1-2 decision points | Partial contract — Routing + Choreography |
| **Heavy** | Multi-deliverable, high stakes, unclear path | Full six-part contract |

Be honest about complexity. Don't over-engineer a simple request — that wastes the user's time and erodes trust. The value of this skill is knowing *when* to go deep, not going deep on everything.

### Step 2: Extract Intent

Before restructuring, identify:

- **Goal**: What does "done" look like? (deliverable, state change, decision)
- **Audience**: Who consumes the output?
- **Constraints**: Time, format, tools, brand, technical stack
- **Unknowns**: What's ambiguous that will cause problems mid-execution
- **Assumed context**: References to "the project", "our approach" — surface and confirm

If critical info is missing, ask 1-2 targeted questions. Make assumptions for non-critical items and state them explicitly.

### Step 3: Build the Structured Prompt

Output a restructured version using the appropriate depth:

- **Light**: 1-2 sentences. Sharpen the ask, fill implied context, specify format and scope. Then execute immediately.
- **Medium**: Compact execution plan with Approach, Steps (including checkpoints), and Done When criteria.
- **Heavy**: Full six-part behavioral contract covering Role, Approach Decision, Execution Flow, Verification, Recovery, and Boundaries.

For complete template formats and worked examples, consult `references/templates.md`.

### Step 4: Execute or Hand Back

After presenting the structured prompt, offer clear options:

1. **"Execute this now?"** → Route to execution (see Execution Handoff below)
2. **"Adjust first?"** → Let them modify before execution
3. **"Save as template?"** → If repeatable, offer to save the pattern

Never dump the restructured prompt and stop. The goal is to accelerate work, not create homework.

## Execution Handoff

When the user approves execution, route correctly based on context:

**If another skill handles the deliverable:**
- Invoke the target skill via the Skill tool — DO NOT replicate its logic inline
- Pass the structured prompt as context to the target skill
- Defer to the target skill's own workflow for execution steps
- The prompt-architect produces the WHAT and HOW. The target skill produces the DELIVERABLE.

**If orchestrating sub-agents:**
- Use the Task tool for spawning agents — DO NOT use bash commands for agent invocation
- Specify complete parameters: `subagent_type`, `prompt` (with full structured context), `model`
- Execute steps sequentially when they depend on prior output — DO NOT parallelize dependent steps
- Each sub-agent receives the relevant section of the structured prompt, not the entire contract

**If self-executing (no other skill needed):**
- Follow the structured prompt's choreography directly
- Hit each checkpoint before proceeding
- Report completion against the verification criteria

## Calibration Guide

**Go lighter than you think for:**
- Tasks the user has done before (they know the path)
- Clearly-scoped requests with specific deliverables
- Time-sensitive requests ("quick", "just need", "fast")

**Go deeper than you think for:**
- First-time tasks in a new domain
- Tasks with "I think", "maybe", "not sure how to approach"
- Multi-audience deliverables (pitch decks, proposals, strategies)
- Tasks that will be iterated on (architecture docs, systems design)
- Anything explicitly asked to "think through" or "plan out"

## Anti-Patterns

- **Over-engineering**: Don't produce a six-part contract for "summarize this PDF." User trust erodes when you add ceremony to simple tasks.
- **Meta-recursion**: Don't prompt-architect the prompt-architect. If asked to refine a prompt, refine it.
- **Paralysis by analysis**: The restructured prompt should make execution *faster*. If your output requires more reading than doing the task, you've failed.
- **Ignoring user style**: Match the user's natural communication style — bullet points, prose, or diagrams.
- **Hiding assumptions**: Every assumption must be visible. The user catches wrong assumptions *before* execution, not after.
- **Inline skill replication**: When a target skill exists for the deliverable, use it. Don't rebuild its logic inside the structured prompt.

## Troubleshooting

### User rejects the structured prompt

**Symptom**: "Just do it" / "This is overkill"

**Cause**: Complexity was over-assessed, or user values speed over structure.

**Recovery**: Immediately drop to Light reframe. Say "Got it — here's the short version:" and execute. Don't defend the process.

### Complexity assessment was wrong

**Symptom**: Mid-execution discovery of major unknowns in a Light/Medium prompt.

**Recovery**: Pause, acknowledge the gap, upgrade to the next complexity tier for the remaining steps. Don't restart from scratch — build on what's done.

### Target skill doesn't exist

**Symptom**: Structured prompt calls for a deliverable type with no matching skill.

**Recovery**: Execute directly using the structured prompt's choreography. Note in output that a dedicated skill would improve consistency for repeat tasks.

### User provides conflicting constraints

**Symptom**: "Make it thorough but keep it to 2 sentences"

**Recovery**: Surface the conflict explicitly. Ask: "Which matters more — completeness or brevity?" Don't silently choose.

### Structured prompt keeps growing

**Symptom**: Contract exceeds what the user asked for, accumulating scope.

**Recovery**: Re-read the original request. Strip anything the user didn't ask for. If you added it because it "seemed important," flag it as optional rather than embedding it in the contract.

## Integration

This skill is an **orchestrator** — it produces execution contracts that other skills consume.

| Upstream (feeds into prompt-architect) | Downstream (prompt-architect feeds into) |
|----------------------------------------|------------------------------------------|
| User's raw request | Any creation skill (component-generator, seo-copy, etc.) |
| Brainstorming output | Writing/planning skills |
| Review/audit findings | Fix/implementation skills |

**Flow**: Entry trigger → prompt-architect assesses and structures → target skill executes deliverable.

The prompt-architect decides WHAT to build and HOW to approach it. Domain skills handle the BUILD.
