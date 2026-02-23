# Skill Discovery Engine

## Overview

Dynamic skill discovery eliminates static configuration. Skills are discovered at runtime by scanning known directories for SKILL.md files.

## Discovery Algorithm

```
FUNCTION discoverSkills():
    skills = []

    // 1. Define search paths
    paths = [
        "~/.claude/skills",           // Global user skills
        "./.claude/skills",           // Project skills
        "{configured_paths}",         // From CLAUDE.md
        "~/.claude/plugins/cache/*/skills"  // Plugin skills
    ]

    // 2. Scan each path
    FOR path IN paths:
        files = glob("{path}/**/SKILL.md")
        FOR file IN files:
            skill = parseSkillFile(file)
            skills.append(skill)

    // 3. Build index
    RETURN buildIndex(skills)
```

## Parsing SKILL.md Files

### Frontmatter Extraction

SKILL.md files use YAML frontmatter:

```markdown
---
name: skill-name
description: What this skill does. Use when X, Y, Z...
---

# Skill Content
...
```

Parse by:
1. Find first `---` (start of frontmatter)
2. Find second `---` (end of frontmatter)
3. Parse YAML between delimiters
4. Content is everything after second `---`

### Trigger Extraction

Triggers are extracted from the `description` field:

```python
def extract_triggers(description: str) -> list[str]:
    triggers = []

    # 1. Find "Use when" section
    if "Use when" in description:
        use_when_text = description.split("Use when")[1]

        # 2. Extract keywords (comma or space separated)
        keywords = re.findall(r'\b\w+\b', use_when_text.lower())

        # 3. Filter to meaningful triggers
        stopwords = {'when', 'the', 'a', 'an', 'or', 'and', 'to', 'for'}
        triggers = [k for k in keywords if k not in stopwords]

    # 4. Also extract from full description
    domain_words = ['api', 'component', 'ui', 'deploy', 'aws', 'test',
                    'lint', 'build', 'validate', 'audit', 'create', 'generate']
    for word in domain_words:
        if word in description.lower():
            triggers.append(word)

    return list(set(triggers))
```

## Skill Index Structure

```typescript
interface SkillEntry {
    name: string;           // From frontmatter
    path: string;           // Full path to SKILL.md
    description: string;    // From frontmatter
    triggers: string[];     // Extracted keywords
    content?: string;       // Loaded on demand
}

interface SkillIndex {
    skills: Map<string, SkillEntry>;
    byTrigger: Map<string, string[]>;  // trigger -> skill names
    lastScanned: Date;
}
```

## Matching Algorithm

```python
def match_skills(task: str, index: SkillIndex) -> list[SkillEntry]:
    task_lower = task.lower()
    task_words = set(re.findall(r'\b\w+\b', task_lower))

    scores = {}
    for skill_name, skill in index.skills.items():
        score = 0

        # Direct trigger matches
        for trigger in skill.triggers:
            if trigger in task_words:
                score += 2  # Exact word match
            elif trigger in task_lower:
                score += 1  # Substring match

        # Boost for domain relevance
        if skill_name in task_lower:
            score += 3

        if score > 0:
            scores[skill_name] = score

    # Sort by score, return top matches
    sorted_skills = sorted(scores.items(), key=lambda x: -x[1])
    return [index.skills[name] for name, _ in sorted_skills if _ >= 2]
```

## Auto-Include Rules

Certain skills should be automatically included based on context:

```python
AUTO_INCLUDE_RULES = {
    # If any of these matched, also include these
    "api-route-generator": ["linting-build-validator"],
    "component-generator": ["linting-build-validator", "design-system-auditor"],
    "aws-cdk-development": ["linting-build-validator"],

    # Always include for code generation tasks
    "_code_generation": ["linting-build-validator"],
}

def apply_auto_includes(matched: list[str]) -> list[str]:
    result = set(matched)
    for skill in matched:
        if skill in AUTO_INCLUDE_RULES:
            result.update(AUTO_INCLUDE_RULES[skill])

    # Check if any code generation happened
    code_gen_skills = {"api-route-generator", "component-generator", "aws-cdk-development"}
    if result & code_gen_skills:
        result.update(AUTO_INCLUDE_RULES["_code_generation"])

    return list(result)
```

## Execution Order Determination

```python
EXECUTION_PHASES = {
    "creation": ["api-route-generator", "component-generator", "aws-cdk-development"],
    "validation": ["linting-build-validator", "design-system-auditor"],
    "deployment": ["secure-deployment-manager"],
    "documentation": ["changelog-generator"],
}

def determine_execution_order(matched: list[str]) -> list[list[str]]:
    phases = []

    # Phase 1: Creation (parallel)
    creation = [s for s in matched if s in EXECUTION_PHASES["creation"]]
    if creation:
        phases.append({"parallel": True, "skills": creation})

    # Phase 2: Validation (sequential after creation)
    validation = [s for s in matched if s in EXECUTION_PHASES["validation"]]
    if validation:
        phases.append({"parallel": False, "skills": validation})

    # Phase 3: Deployment (if explicitly requested)
    deployment = [s for s in matched if s in EXECUTION_PHASES["deployment"]]
    if deployment:
        phases.append({"parallel": False, "skills": deployment})

    return phases
```

## Lazy Content Loading

Skill content is loaded only when needed:

```python
def load_skill_content(skill: SkillEntry) -> str:
    if skill.content is None:
        with open(skill.path, 'r') as f:
            full_content = f.read()

        # Skip frontmatter
        parts = full_content.split('---')
        if len(parts) >= 3:
            skill.content = '---'.join(parts[2:]).strip()
        else:
            skill.content = full_content

    return skill.content
```

## Adding New Skills

To add a new skill that's automatically discoverable:

1. Create directory: `skills/global/my-skill/`
2. Create `SKILL.md` with proper frontmatter:

```markdown
---
name: my-skill
description: Short description. Use when doing X, creating Y, building Z, or handling W.
---

# My Skill

## When to Use
- Scenario A
- Scenario B

## How It Works
...
```

3. **Done.** Next orchestration will discover it automatically.

## Cache Considerations

For performance, consider caching:

```python
CACHE_TTL = 300  # 5 minutes

class SkillCache:
    def __init__(self):
        self.index = None
        self.last_scan = None

    def get_index(self) -> SkillIndex:
        now = time.time()
        if self.index is None or (now - self.last_scan) > CACHE_TTL:
            self.index = discover_skills()
            self.last_scan = now
        return self.index
```

However, for Claude Code usage, fresh discovery is preferred since:
- Skills don't change mid-conversation often
- Discovery is fast (just file globbing + parsing)
- Ensures new skills are immediately available
