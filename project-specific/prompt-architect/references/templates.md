# Prompt Architect Templates

Reference templates for the three complexity levels. See SKILL.md for when to use each.

## Light Reframe

Just sharpen what they asked. No ceremony.

**Format**: 1-2 sentences that fill implied context, specify format, and define scope.

**Example**:
- Raw: "Make me a doc about our pricing"
- Reframed: "Create a Word document covering Outdoor Butler's pricing structure for pressure washing and artificial grass cleaning services, targeting expat homeowners in Costa Blanca South. Include service tiers, seasonal pricing, and a comparison table."

The reframe fills in the implied context, specifies the format, and defines scope. Then execute immediately — no need for a six-part contract on a single document.

---

## Medium Structure

For tasks with decision points or multiple steps. Output as a compact execution plan:

~~~markdown
## Task: [Clear 1-line description]

### Approach
[Which method/tool/framework to use and why — the routing decision]

### Steps
1. [First action] → produces [intermediate output]
2. [Second action] → depends on output of step 1
3. **CHECKPOINT**: [Verify X before continuing]
4. [Continue...]

### Done When
- [Specific measurable completion criteria]
~~~

### Medium Example

> ## Task: SEO content strategy for Outdoor Butler
>
> ### Approach
> Competitive analysis → keyword gap identification → content calendar. Using web search for competitor analysis, cross-referenced with existing site structure.
>
> ### Steps
> 1. Audit top 5 competitors for "pressure washing Costa Blanca" and related terms
> 2. Map their content against yours — identify gaps
> 3. **CHECKPOINT**: Review gap list before building calendar
> 4. Build 90-day content calendar prioritized by search volume x competition score
> 5. Write brief for each content piece (target keyword, angle, word count)
>
> ### Done When
> - Content calendar covers 12+ weeks
> - Each entry has target keyword, format, and brief
> - Priorities backed by actual search data, not assumptions

---

## Heavy Contract

For complex, high-stakes, or multi-deliverable tasks. Output the full six-part behavioral contract:

~~~markdown
## Task: [Clear 1-line description]

### 1. Role & Expertise Required
[What knowledge/perspective is needed — e.g., "Act as a telecom solutions architect
familiar with TMF OpenAPIs and MCP server patterns"]

### 2. Approach Decision
| If the task involves... | Then... |
|------------------------|---------|
| [Variant A] | [Approach A] |
| [Variant B] | [Approach B] |

Selected approach: [X] because [reason]

### 3. Execution Flow
1. **Discovery**: [What to research/read/check first]
2. **Structure**: [Create skeleton/outline/schema]
3. **Build**: [Fill in content, section by section]
   - CHECKPOINT after [milestone]: validate [specific thing]
4. **Polish**: [Formatting, consistency, edge cases]
5. **Validate**: [Run checks, verify completeness]
6. **Deliver**: [Output format, location, presentation]

### 4. Verification
- [ ] [Specific quality check 1]
- [ ] [Specific quality check 2]
- [ ] [Completeness check: all sections/requirements addressed]

### 5. If Things Go Wrong
- If [common failure mode] → [recovery action]
- If [ambiguity discovered mid-task] → [decision rule or escalate to user]
- After 3 failed attempts at [X] → stop and report specifics

### 6. Boundaries
- Do NOT [anti-pattern specific to this task]
- Do NOT [scope creep risk]
- [Any format/style constraints]
~~~

### Heavy Example (summarized)

For "build the full architecture for the Intelligence Platform's billing deviation detection system":

- **Role**: Telecom solutions architect familiar with TMF OpenAPIs and MCP server patterns
- **Routing**: Decision table between real-time stream processing vs batch analysis, selected based on data volume and latency requirements
- **Execution**: Research phase (TMF standards review) → schema design → validation against TMF624/TMF635 → implementation plan with checkpoints after each phase
- **Verification**: Schema validates against TMF specs, all billing anomaly types covered, recovery paths defined, performance benchmarks specified
- **Recovery**: Data quality issues → fallback to batch processing; API rate limits → circuit breaker pattern; ambiguous anomaly classification → escalate to user with evidence
- **Boundaries**: Don't scope-creep into fraud detection; don't redesign existing rating engine; stay within the billing domain
