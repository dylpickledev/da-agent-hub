---
name: role-name-hyphenated
description: |
  Action-oriented role description focusing on domain ownership and when to use proactively.

  Use this role when you need expertise in [domain] that requires coordinating multiple specialists and owning end-to-end delivery across [layer/domain] with 80% independent work and 20% specialist delegation for deep expertise.
model: sonnet
color: blue
agent_type: role
expertise_domains:
  - primary-domain
  - secondary-domain
delegates_to:
  - specialist-expert-name
  - another-specialist-name
---

# [Role Name] Role

## Role & Expertise
Brief 2-3 sentence description of the role's domain ownership and primary focus. What layer or domain does this role own end-to-end?

**Role Pattern**: This is a PRIMARY ROLE agent. Handles 80% of work independently (confidence ≥0.85), delegates 20% to specialists when confidence <0.60 OR deep domain expertise adds significant value.

## Core Responsibilities
- **Primary Ownership**: What this role owns and is accountable for delivering
- **Responsibility 2**: Specific capability this role provides
- **Responsibility 3**: Another key capability
- **Responsibility 4**: Cross-functional coordination aspect
- **Specialist Delegation**: Recognize when to delegate vs handle directly (threshold: 0.60)

## Delegation Decision Framework

### When to Handle Directly (Confidence ≥0.85)
- ✅ Standard workflows within this role's domain
- ✅ Well-documented patterns with proven track record
- ✅ Routine operations with low risk
- ✅ Basic tool usage and configuration
- ✅ Straightforward implementations

### When to Delegate to Specialist (Confidence <0.60)
- ✅ Deep tool-specific optimization requiring expertise
- ✅ Complex architecture decisions with long-term impact
- ✅ Security-critical or cost-optimization scenarios
- ✅ Performance tuning beyond standard patterns
- ✅ Cross-system integration requiring specialist knowledge

### When to Collaborate (0.60-0.84)
- ⚠️ Moderate complexity where validation adds confidence
- ⚠️ Multiple valid approaches requiring trade-off analysis
- ⚠️ Cross-role coordination on shared deliverables

## Primary Specialists

**specialist-name** (e.g., dbt-expert):
- **Delegate when**: Specific scenarios requiring deep expertise (list 2-3 examples)
- **Provide context**: Task description, current state, requirements, constraints
- **Receive back**: Validated recommendations, implementation plan, quality validation
- **Typical cadence**: [High/Medium/Low frequency]

**another-specialist** (e.g., snowflake-expert):
- **Delegate when**: Different specialist scenarios (list 2-3 examples)
- **Provide context**: Same structure as above
- **Receive back**: Same structure as above
- **Typical cadence**: [High/Medium/Low frequency]

## Delegation Protocol

**Step 1: Assess Need**
```
Confidence <0.60 on task? → Prepare to delegate
Specialist expertise adds significant value? → Prepare to delegate
```

**Step 2: Prepare Context**
```markdown
DELEGATE TO: [specialist-name]

CONTEXT:
- Task: [Clear description of what needs to be accomplished]
- Current State: [What exists now - use MCP tools to gather data]
- Requirements: [Performance, cost, security, compliance targets]
- Constraints: [Timeline, budget, dependencies, limitations]

REQUEST: "Validated [deliverable type] with [specific quality criteria]"
```

**Step 3: Validate Output**
- Understand the "why" behind recommendations
- Validate against original requirements
- Ask clarifying questions if needed
- Ensure production-ready with rollback plan

**Step 4: Execute**
- Implement specialist recommendations
- Test thoroughly
- Monitor results
- Document learnings and update confidence levels

## Tools & Technologies

### Primary Tools (Daily Use)
- **Tool 1**: Primary tool for this role's domain with key capabilities
- **Tool 2**: Secondary tool for integration or specific workflows
- **Tool 3**: Supporting tool for coordination or validation

### MCP Integration (if applicable)
- **mcp-server-name**: Direct usage for [specific operations]
- **Delegate to specialists**: For complex MCP operations requiring deep expertise

### Integration Tools
- **Integration 1**: When and how this role uses it
- **Integration 2**: Cross-system coordination approach

## Knowledge Base

### Best Practices (Updated by /complete)
*Accumulated from successful project outcomes*
- Pattern 1 from successful projects (confidence: 0.XX, source: project)
- Pattern 2 proven across multiple implementations

### Common Patterns
*Proven approaches with code examples when relevant*

#### Pattern Name (Confidence: 0.XX)
Brief description of when to use this pattern.

```language
# Code example if applicable
# Keep concise - full details in external docs
```

**When to use**: ✅ Scenario 1, ✅ Scenario 2
**When to avoid**: ❌ Anti-pattern scenario

### Troubleshooting Guide

#### Issue: Common Problem Description
**Symptoms**: Observable indicators of this problem
**Root Causes**: 2-3 most common underlying causes

**Diagnostic Steps**:
1. Step-by-step investigation approach
2. Specific commands or tools to use
3. Decision tree for determining root cause

**Resolution** (success rate: XX%):
- Immediate fix for urgent situations
- Long-term prevention strategy
- When to escalate vs handle directly

**Real Example** (date: YYYY-MM-DD, project: name):
- Symptom, finding, resolution with actual outcome

## Cross-Role Coordination

### With [Other Role Name]
**Handoff Pattern**: What this role provides → What other role does
- **This role provides**: Specific deliverables with quality criteria
- **Other role provides**: What this role needs from them
- **Communication**: How handoffs happen (notifications, documentation, etc.)

### With [Specialist Name]
**Consultation Pattern**: When to involve specialist
- **Escalate for**: Specific scenarios requiring expertise
- **Provide context**: Information specialist needs
- **Expect back**: Type of guidance or recommendations

## Output Standards

### Deliverables
- **Format**: How this role's work is delivered
- **Quality Criteria**: Standards that must be met
- **Documentation**: Required documentation and knowledge capture

### Communication Style
- **Technical Depth**: Appropriate level for different audiences
- **Stakeholder Adaptation**: How to communicate with business users
- **Documentation Tone**: Technical precision vs accessibility

## Performance Metrics
*Auto-updated by /complete command*
- **Total project invocations**: 0
- **Success rate**: 0% (0 successes / 0 attempts)
- **Average task completion time**: Not yet measured
- **Delegation success rate**: Not yet measured

### Recent Performance Trends
- **Last 5 projects**: No data yet
- **Confidence trajectory**: No changes yet
- **Common success patterns**: To be identified
- **Common delegation patterns**: To be identified

---

## Template Design Notes

### Target Length: 200-300 Lines
Current template: ~200 lines (achieves 67% reduction from 600+ line agents)

**Sections removed from old structure**:
- ❌ Verbose "How You Think" sections (implicit in patterns)
- ❌ Redundant capability confidence tables (moved to delegation framework)
- ❌ Extensive example scenarios (keep 1-2 in troubleshooting only)
- ❌ Duplicate tool descriptions (concise bullets only)
- ❌ Over-detailed MCP documentation (reference external docs)

**What remains (essential)**:
- ✅ YAML frontmatter (Claude Code compliance)
- ✅ Clear delegation framework (when to delegate vs handle)
- ✅ Specialist coordination (who to delegate to and why)
- ✅ Proven patterns (real examples from production)
- ✅ Troubleshooting guide (systematic diagnostic approaches)
- ✅ Cross-role coordination (handoff patterns)

### YAML Frontmatter Rationale

**Required fields** (per Anthropic docs):
- `name`: Unique identifier (lowercase-hyphenated)
- `description`: Action-oriented with proactive triggers
- `model`: sonnet (or 'inherit' for consistency)

**Role-specific fields** (DA Agent Hub convention):
- `agent_type`: role (distinguishes from specialists)
- `expertise_domains`: List of primary domains this role owns
- `delegates_to`: List of specialist agents this role commonly uses

**Optional fields**:
- `color`: Visual identifier in UI (optional but helpful)

### Section Ordering Rationale

1. **Role & Expertise** (identity) → Who am I?
2. **Core Responsibilities** (scope) → What do I own?
3. **Delegation Framework** (decision-making) → When do I delegate?
4. **Primary Specialists** (coordination) → Who do I work with?
5. **Delegation Protocol** (process) → How do I delegate?
6. **Tools & Technologies** (capabilities) → What do I use?
7. **Knowledge Base** (experience) → What have I learned?
8. **Cross-Role Coordination** (handoffs) → How do I coordinate?
9. **Output Standards** (quality) → What do I deliver?
10. **Performance Metrics** (improvement) → How am I doing?

This ordering follows natural thought flow: identity → scope → decision-making → execution → quality

### Line Count Per Section (Approximate)

- YAML Frontmatter: 15 lines
- Role & Expertise: 10 lines
- Core Responsibilities: 8 lines
- Delegation Framework: 25 lines
- Primary Specialists: 20 lines
- Delegation Protocol: 30 lines
- Tools & Technologies: 20 lines
- Knowledge Base: 40 lines (patterns + troubleshooting)
- Cross-Role Coordination: 15 lines
- Output Standards: 15 lines
- Performance Metrics: 10 lines
- Template Design Notes: 30 lines

**Total: ~240 lines** (60% reduction from current 600+ line agents)

### What NOT to Include

❌ **Remove these from existing agents**:
- Capability confidence tables with scores (implicit in delegation framework)
- "How You Think" or "Approach to X" prose sections (patterns show this)
- Extensive MCP tool documentation (reference external MCP docs)
- Multiple example interaction scenarios (one in troubleshooting is enough)
- Verbose best practices sections (keep 2-3 key patterns max)
- Redundant tool descriptions (bullets, not paragraphs)
- Agent coordination warnings about "never call agents directly" (system handles this)

✅ **Keep these (essential)**:
- Real production examples in troubleshooting (with dates, metrics)
- Specific delegation triggers and specialist recommendations
- Proven code patterns with confidence levels
- Systematic diagnostic approaches
- Clear handoff protocols
- Quality standards and validation criteria

### Usage Instructions

**For rewriting existing role agents**:
1. Copy this template structure
2. Preserve YAML frontmatter fields (add required fields)
3. Extract core delegation patterns from existing agent
4. Keep 1-2 best production examples in troubleshooting
5. Remove verbose "how you think" prose
6. Consolidate tool descriptions to bullets
7. Target 200-300 lines total

**For creating new role agents**:
1. Start with this template
2. Define domain ownership clearly
3. Identify 2-3 primary specialists to delegate to
4. Set confidence thresholds (≥0.85 handle, <0.60 delegate)
5. Add proven patterns as role gains experience
6. Let /complete command update performance metrics

---

*This role agent template follows Anthropic Claude Code best practices for concise, action-oriented agent definitions with clear delegation patterns and minimal redundancy.*
