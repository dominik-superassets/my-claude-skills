# My Claude Code Skills

Personal collection of Claude Code skills organized by source.

## Directory Structure

```
my-claude-skills/
├── global/              # Custom skills (available globally via ~/.claude/skills/)
├── marketplace/
│   ├── superpowers/     # Superpowers plugin skills
│   └── example-skills/  # Anthropic example skills
└── project-specific/    # Project-specific skills (reference only)
```

## Quick Setup

Skills must be installed to your Claude Code skills directory to be available.

**Skills directory location:**
```bash
~/.claude/skills/          # Linux/Mac
C:\Users\<you>\.claude\skills\   # Windows
```

**Easiest installation method:** Ask Claude Code to install skills:

```
"Install the skill-orchestrator skill from this repo to my Claude skills directory"
```

Claude Code will detect your skills directory and copy the skill folder.

**Manual installation:**
```bash
cp -r global/<skill-name> ~/.claude/skills/
```

**Verify installation:** After installing, the skill should appear when you type `/` in Claude Code.

---

## Global Skills (Custom)

These are your custom skills available in all Claude Code sessions.

| Skill | Description |
|-------|-------------|
| **api-route-generator** | Create Next.js 15 API routes with authentication, permission checks, request validation, error handling |
| **aws-cdk-development** | AWS CDK expert for building cloud infrastructure with TypeScript/Python |
| **aws-cost-operations** | AWS cost optimization, monitoring, and operational best practices |
| **aws-infrastructure-manager** | Comprehensive AWS infrastructure management using aws-dev and aws-prod MCP servers |
| **aws-serverless-eda** | AWS serverless and event-driven architecture expert based on Well-Architected Framework |
| **changelog-generator** | Auto-creates user-facing changelogs from git commits |
| **component-generator** | Generate React TypeScript components following SynthCore's design system |
| **design-system-auditor** | Design system compliance scanner detecting violations |
| **linting-build-validator** | Pre-commit and pre-deployment validation for linting and TypeScript |
| **ralph-wiggum-prompts** | Transform plans/tasks into Ralph Wiggum loop-ready prompts |
| **secure-deployment-manager** | Deploy to AWS Amplify with security-first debugging |
| **skill-orchestrator** | **META-SKILL** - Enables skill-augmented sub-agent spawning for scale autonomy |

---

## Skill Orchestrator (Scale Autonomy)

The `skill-orchestrator` is a meta-skill that enables **recursive, skill-aware agent spawning** with dynamic discovery.

### Hybrid Approach: Native + Orchestrator

Claude Code supports **two complementary** patterns for skill-augmented agents:

| Pattern | When to Use |
|---------|-------------|
| **Native `skills:` frontmatter** | Fixed specialist agents (always same skills) |
| **skill-orchestrator** | Dynamic tasks, workflows, auto-matching |

**Use both together** for maximum flexibility.

### Native Custom Agents (Static Binding)

For roles that always need the same expertise:

```yaml
# .claude/agents/api-specialist.md
---
name: api-specialist
description: API development with conventions loaded
tools: [Read, Write, Edit, Bash, Glob, Grep]
skills: [api-route-generator, linting-build-validator]
---
You are an API specialist. Create endpoints following loaded skill patterns.
```

Invoke with: `Task tool → subagent_type: api-specialist`

### skill-orchestrator (Dynamic Matching)

For tasks where skills should be auto-discovered:

```
User: "Build a preferences feature"
    ↓
Orchestrator scans for SKILL.md files
    ↓
Matches: api-route-generator + component-generator
    ↓
Spawns parallel agents with skill knowledge injected
    ↓
Chains to linting-build-validator automatically
    ↓
Reports complete feature with validated artifacts
```

### What skill-orchestrator Provides (Beyond Native)

| Capability | Native | Orchestrator |
|------------|--------|--------------|
| Skill content → subagent | ✅ | ✅ |
| Works with standard Task tool | ❌ | ✅ |
| Dynamic skill discovery | ❌ | ✅ |
| Auto-matching from task | ❌ | ✅ |
| Workflow pipelines (YAML) | ❌ | ✅ |
| Variable passing | ❌ | ✅ |
| Refinement loops | ❌ | ✅ |

### Key Files

```
global/skill-orchestrator/
├── SKILL.md                           # Core orchestration instructions
├── workflows/                         # Declarative YAML pipelines
│   ├── full-feature.yaml              # End-to-end feature creation
│   ├── validate-and-deploy.yaml       # Pre-flight + deployment
│   └── quick-validate.yaml            # Fast pre-commit checks
└── references/
    ├── skill-discovery.md             # Dynamic discovery algorithm
    ├── workflow-schema.md             # Workflow YAML specification
    ├── workflow-executor.md           # Execution engine details
    └── skill-mesh.json                # Legacy registry (deprecated by discovery)
```

### Workflow Example

```yaml
# workflows/full-feature.yaml
name: Full Feature Pipeline
trigger: "build feature *"
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

### Output Contract

All skill-augmented agents return structured JSON:
```json
{
  "status": "complete | needs_refinement | blocked",
  "result": "what was accomplished",
  "artifacts": ["files created/modified"],
  "request_skills": ["additional skills needed"],
  "spawn_requests": [{ "task": "...", "suggested_skills": [...] }]
}
```

---

## Marketplace Skills

### Superpowers Plugin

Workflow and development methodology skills.

| Skill | Purpose |
|-------|---------|
| **brainstorming** | Explore user intent before creative work |
| **dispatching-parallel-agents** | Run 2+ independent tasks in parallel |
| **executing-plans** | Execute written implementation plans with checkpoints |
| **finishing-a-development-branch** | Complete development work - merge, PR, or cleanup |
| **receiving-code-review** | Handle code review feedback properly |
| **requesting-code-review** | Request review when completing tasks |
| **subagent-driven-development** | Execute plans with fresh subagent per task |
| **systematic-debugging** | Debug bugs/failures before proposing fixes |
| **test-driven-development** | TDD - write tests first, watch them fail |
| **using-git-worktrees** | Create isolated git worktrees for feature work |
| **using-superpowers** | Find and use skills effectively |
| **verification-before-completion** | Verify work before claiming complete |
| **writing-plans** | Write comprehensive implementation plans |
| **writing-skills** | Create and edit skills |

### Anthropic Example Skills

Document and media creation skills.

| Skill | Purpose |
|-------|---------|
| **algorithmic-art** | Create generative art with p5.js |
| **brand-guidelines** | Apply Anthropic brand colors and typography |
| **canvas-design** | Create visual art as PNG/PDF |
| **doc-coauthoring** | Co-author documentation through structured workflow |
| **docx** | Create/edit Word documents with tracked changes |
| **frontend-design** | Create production-grade frontend interfaces |
| **internal-comms** | Write internal communications |
| **mcp-builder** | Build MCP servers for LLM integrations |
| **pdf** | PDF manipulation - extract, create, merge, split |
| **pptx** | Create/edit PowerPoint presentations |
| **skill-creator** | Guide for creating effective skills |
| **slack-gif-creator** | Create animated GIFs for Slack |
| **theme-factory** | Style artifacts with themes |
| **webapp-testing** | Test web apps with Playwright |
| **web-artifacts-builder** | Create multi-component HTML artifacts |
| **xlsx** | Create/edit spreadsheets with formulas |

---

## Usage

### Invoking Skills

Skills trigger automatically based on context, or can be invoked:

```
/skill-name
```

### Adding New Skills

1. Create skill directory with `SKILL.md`
2. Add to `~/.claude/skills/` for global availability
3. Update this catalog

### Skill Structure

```
skill-name/
├── SKILL.md           # Required - frontmatter + instructions
└── references/        # Optional - additional docs
```

---

## Locations Reference

| Location | Purpose |
|----------|---------|
| `~/.claude/skills/` | Global skills (all sessions) |
| `<project>/.claude/skills/` | Project-specific skills |
| `~/.claude/plugins/cache/*/skills/` | Marketplace plugin skills |

---

*Last updated: 2025-01-12*
