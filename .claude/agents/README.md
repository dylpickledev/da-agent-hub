# DA Agent Hub: Agent Architecture Guide

## Overview

This document explains the **agent architecture** in DA Agent Hub - how role agents, specialist agents, skills, and MCP tools work together to provide effective AI assistance for data and analytics work.

**Architecture Pattern**: Role Agents (80% independent) → Specialist Agents (20% consultation)
**Priority**: Correctness > Speed
**Foundation**: Anthropic Claude Code best practices

---

## Quick Reference

**Current Agents (7 total)**:
- **Role Agents** (3): analytics-engineer-role, data-engineer-role, onboarding-agent
- **Specialist Agents** (4): dbt-expert, snowflake-expert, tableau-expert, claude-code-expert

**Skills** (4): project-setup, pr-description-generator, dbt-model-scaffolder, documentation-validator

**MCP Servers** (1): dbt (dbt Cloud + Semantic Layer access)

---

## Architecture Layers

```
┌─────────────────────────────────────────────────────────┐
│ LAYER 1: ROLE AGENTS (Primary - 80% of Work)           │
│                                                         │
│ ✅ analytics-engineer-role                             │
│    - dbt transformations, data modeling, SQL           │
│    - Delegates to: dbt-expert, snowflake-expert        │
│                                                         │
│ ✅ data-engineer-role                                  │
│    - Data pipelines, orchestration, ingestion          │
│    - Delegates to: snowflake-expert                    │
│    - Creates specialists as needed for tools           │
│                                                         │
│ ✅ onboarding-agent                                    │
│    - Setup, troubleshooting, workflow education        │
│    - Delegates to: dbt-expert, claude-code-expert      │
│                                                         │
│ What they do: Own end-to-end workflows                 │
│ When they delegate: Confidence <0.60 OR need expertise │
└─────────────────────────────────────────────────────────┘
                          ↓
              DELEGATES TO SPECIALISTS
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LAYER 2: SPECIALIST AGENTS (Consultation - 20%)        │
│                                                         │
│ Data Platform Specialists:                             │
│ ✅ dbt-expert - dbt patterns, macros, optimization     │
│ ✅ snowflake-expert - Warehouse cost/performance       │
│ ✅ tableau-expert - BI dashboard optimization          │
│                                                         │
│ Platform Specialists:                                  │
│ ✅ claude-code-expert - Agent/skill configuration      │
│                                                         │
│ What they do: Deep expertise + validated solutions     │
│ How they work: Domain knowledge + MCP tools            │
└─────────────────────────────────────────────────────────┘
                          ↓
                USE MCP TOOLS + EXPERTISE
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LAYER 3: MCP TOOLS (Data Access)                       │
│                                                         │
│ Currently Configured:                                   │
│ ✅ dbt - dbt Cloud API + Semantic Layer                │
│    - Used by: dbt-expert, analytics-engineer-role      │
│    - Provides: Model details, job status, queries      │
│                                                         │
│ What MCP provides: Real-time data access               │
│ What MCP does NOT provide: Expertise or decisions      │
└─────────────────────────────────────────────────────────┘
```

---

## Agent Types: Roles vs Specialists

### Role Agents (Primary Agents)

**Purpose**: Own end-to-end workflows within a domain (80% of work independently)

**Characteristics**:
- Broad domain ownership (e.g., all transformation work, all pipeline work)
- Handle common tasks directly (confidence ≥0.85)
- Delegate to specialists when needed (confidence <0.60)
- Create new specialists using templates when encountering new tools

**Current Role Agents**:

1. **analytics-engineer-role** (`analytics-engineer-role.md`)
   - Domain: dbt transformations, data modeling, SQL optimization
   - Delegates to: dbt-expert (complex macros), snowflake-expert (cost optimization)
   - Tools: dbt-mcp for model management

2. **data-engineer-role** (`data-engineer-role.md`)
   - Domain: Data pipelines, orchestration, source integration
   - Delegates to: snowflake-expert (loading optimization)
   - Creates specialists: For Orchestra, dlthub, AWS when needed

3. **onboarding-agent** (`onboarding-agent.md`)
   - Domain: User setup, MCP configuration, workflow education
   - Delegates to: dbt-expert (dbt-specific help), claude-code-expert (customization)
   - Tools: Guides users through MCP setup and validation

**Creating New Role Agents**:
- Use template: `.claude/agents/roles/role-template.md`
- Include YAML frontmatter (required for Claude Code)
- Define delegation thresholds (when to handle vs delegate)

### Specialist Agents (Consultation Agents)

**Purpose**: Provide deep expertise in specific tools/domains (20% consultation for complex cases)

**Characteristics**:
- Deep expertise in one tool or domain
- Use MCP tools + domain knowledge to validate recommendations
- Research-focused (analyze, recommend, validate - not execute)
- Called by role agents when confidence <0.60

**Current Specialist Agents**:

1. **dbt-expert** (`specialists/dbt-expert.md`)
   - Expertise: dbt macros, incremental strategies, testing frameworks
   - MCP tools: dbt-mcp (full access)
   - Called by: analytics-engineer-role, onboarding-agent

2. **snowflake-expert** (`specialists/snowflake-expert.md`)
   - Expertise: Warehouse cost optimization, query performance
   - MCP tools: dbt-mcp (for queries via dbt show)
   - Called by: analytics-engineer-role, data-engineer-role

3. **tableau-expert** (`specialists/tableau-expert.md`)
   - Expertise: Dashboard performance, BI optimization
   - MCP tools: None currently (uses Read/WebFetch)
   - Called by: When BI work needed

4. **claude-code-expert** (`specialists/claude-code-expert.md`)
   - Expertise: Agent configuration, skills, Claude Code best practices
   - MCP tools: None (uses Read/WebFetch for docs)
   - Called by: onboarding-agent, when customizing framework

**Creating New Specialist Agents**:
- Use template: `.claude/agents/specialists/specialist-template.md`
- Include YAML frontmatter (required)
- Define MCP tool usage and expertise boundaries

---

## Agent vs Skill: When to Use Each

**Use AGENTS for** (expertise and judgment):
- ✅ Complex analysis requiring domain expertise
- ✅ Troubleshooting that needs diagnosis
- ✅ Decisions with multiple valid approaches
- ✅ Tool-specific optimization
- **Example**: "Optimize this slow dbt model" → dbt-expert (requires analysis)

**Use SKILLS for** (procedural automation):
- ✅ Repeatable multi-step workflows
- ✅ Template-driven generation
- ✅ Checklist-based validation
- ✅ Process automation
- **Example**: "Create dbt model boilerplate" → dbt-model-scaffolder skill (follows template)

**See**: `docs/agents-vs-skills-analysis.md` for comprehensive guide

---

## Delegation Pattern

### When Role Agents Delegate

**Confidence Thresholds**:
- **≥0.85**: Handle directly (80% of work)
- **0.60-0.84**: Collaborate or validate with specialist
- **<0.60**: Delegate to specialist (20% of work)

**Delegation Protocol**:

```markdown
DELEGATE TO: [specialist-name]

CONTEXT:
- Task: [What needs to be accomplished]
- Current State: [Existing configuration, metrics]
- Requirements: [Performance, cost, quality targets]
- Constraints: [Timeline, dependencies]

REQUEST: "Validated [deliverable] with [quality criteria]"
```

### MCP Tool Usage

**Role agents** can use MCP directly for simple operations:
- analytics-engineer-role: Uses dbt-mcp for model details, list jobs
- data-engineer-role: Uses dbt show for data validation

**Specialist agents** use MCP + expertise for complex analysis:
- dbt-expert: Full dbt-mcp access for optimization analysis
- snowflake-expert: Uses dbt show (via dbt-mcp) for query analysis

**Pattern**: MCP provides data → Specialist provides expertise → Role executes

---

## Current Capabilities

### What's Available Now

**Agents** (7 total):
- 3 role agents for primary workflows
- 4 specialist agents for deep expertise

**MCP Integration** (1 server):
- dbt Cloud API via dbt-mcp
- No Snowflake direct access (uses dbt show)
- No AWS, GitHub, or other MCP servers configured

**Skills** (4 procedural workflows):
- project-setup, pr-description-generator, dbt-model-scaffolder, documentation-validator

### What to Add When Needed

**For your specific use case, create**:
- **Additional MCP servers**: Install from community or create custom
- **New role agents**: Use role-template.md for your domain
- **New specialists**: Use specialist-template.md for tool expertise
- **More skills**: For your repeatable workflows

**MCP servers you might add**:
- Snowflake MCP (if direct Snowflake access needed)
- GitHub MCP (for repository operations)
- Custom MCP for your internal tools

---

## Why This Architecture?

### Research-Backed Design

Based on Anthropic guidance:

1. **"Multi-agent systems provide significantly better outcomes"**
   - Specialist consultation prevents costly errors
   - 15x token cost justified by correctness

2. **"MCP tools are data access, not expertise"**
   - MCP tells you "what is" (current state)
   - Specialists tell you "what should be" (recommendations)

3. **"Matches real-world team structures"**
   - Generalist → Specialist consultation is proven pattern
   - Mirrors how human teams work

### Benefits

**Correctness**:
- Specialist validation prevents production errors
- Domain expertise improves decision quality

**Scalability**:
- Easy to add new specialists for new tools
- Role agents don't need to know every tool deeply

**Maintainability**:
- Clear separation of concerns
- Easy to update specialist expertise without touching role agents

---

## File Structure

```
.claude/agents/
├── README.md                          # This file
├── roles/                             # Role agents (primary)
│   ├── analytics-engineer-role.md
│   ├── data-engineer-role.md
│   ├── onboarding-agent.md
│   ├── role-template.md               # Template for new roles
│   └── TEMPLATE-DESIGN-RATIONALE.md   # Design decisions
├── specialists/                       # Specialist agents (consultation)
│   ├── dbt-expert.md
│   ├── snowflake-expert.md
│   ├── tableau-expert.md
│   ├── claude-code-expert.md
│   └── specialist-template.md         # Template for new specialists
```

**Agent Format** (required YAML frontmatter):
```yaml
---
name: agent-name
description: |
  Multi-line description
model: sonnet
color: blue
agent_type: role  # or specialist
expertise_domains: [domain1, domain2]
delegates_to: [specialist1, specialist2]
---
```

---

## Quick Start

### Using Existing Agents

**For dbt work**:
```bash
# Use role agent for general dbt work
claude "Create a new customer dimension model"
# → analytics-engineer-role handles it

# Use specialist for complex optimization
claude "Optimize this incremental model"
# → analytics-engineer-role delegates to dbt-expert
```

**For pipeline work**:
```bash
# Use role agent for standard pipelines
claude "Set up Salesforce data ingestion"
# → data-engineer-role handles it

# Creates specialist if needed for complex tools
# → data-engineer-role might create dlthub-expert
```

**For setup help**:
```bash
# Onboarding agent helps new users
claude "Help me set up MCP"
# → onboarding-agent guides through setup
```

### Creating New Agents

**New role agent**:
1. Copy `.claude/agents/roles/role-template.md`
2. Customize for your domain
3. Define delegation thresholds
4. Test with `claude --agent your-role-name`

**New specialist**:
1. Copy `.claude/agents/specialists/specialist-template.md`
2. Define expertise and MCP tool usage
3. Document when role agents should delegate
4. Test delegation pattern

---

## Further Reading

- **Agent vs Skill**: `docs/agents-vs-skills-analysis.md`
- **Template Design**: `.claude/agents/roles/TEMPLATE-DESIGN-RATIONALE.md`
- **Claude Code Docs**: https://docs.claude.com/en/docs/claude-code/agents
- **MCP Setup**: `docs/troubleshooting-mcp.md`
