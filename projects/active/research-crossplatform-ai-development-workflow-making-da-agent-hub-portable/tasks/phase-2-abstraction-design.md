# Phase 2: Abstraction Design & Tool-Agnostic Architecture

**Project**: Cross-Platform AI Development Workflow
**Phase**: 2 - Abstraction Design
**Date**: 2025-10-16
**Status**: 🚧 In Progress

---

## Executive Summary

This phase defines the **tool-agnostic abstractions** that enable the DA Agent Hub ADLC workflow to run on any AI coding tool. Based on Phase 1 findings, we're designing:

1. **Universal Workflow Interface**: Standard command signatures and behaviors
2. **Portable Agent Format (v2)**: Tool-invocation-agnostic agent definitions
3. **Adapter Pattern**: Tool-specific thin wrappers around portable core
4. **Coordination Protocols**: Agent collaboration without Claude's Task tool

**Core Design Principle**: **80% shared (portable core) + 20% adapted (tool wrapper)**

---

## 1. Tool-Agnostic Workflow Interface Specification

### Universal Command Interface (UCI)

All DA Agent Hub implementations must provide these commands with consistent behavior:

#### Command Signatures

```yaml
# Universal Command Interface Specification v1.0

commands:
  research:
    signature: "research <text|issue-number>"
    purpose: "Deep exploratory analysis for fuzzy ideas"
    entry_point: true
    requires_context: false
    outputs:
      - research project in projects/active/research-*/
      - comprehensive analysis document
      - optional GitHub issue if text provided

  capture:
    signature: "capture <idea-text>"
    purpose: "Quick idea storage with GitHub issue creation"
    entry_point: true
    requires_context: false
    outputs:
      - GitHub issue with 'idea' label
      - auto-categorization labels
      - optional idea fleshing-out conversation

  start:
    signature: "start <issue-number|idea-text>"
    purpose: "Begin development (from issue or ad-hoc)"
    entry_point: true
    requires_context: false
    outputs:
      - project structure in projects/active/feature-*/
      - GitHub issue created (if text) or linked (if issue#)
      - worktree created (if configured)
      - 'in-progress' label added to issue

  switch:
    signature: "switch [branch-name]"
    purpose: "Context switch with work preservation"
    entry_point: false
    requires_context: true  # Must have active work
    outputs:
      - current work committed and pushed
      - switched to specified branch or main
      - clean working directory

  pause:
    signature: "pause [description]"
    purpose: "Save conversation state for resumption"
    entry_point: false
    requires_context: true  # Must have active conversation
    outputs:
      - conversation context saved
      - resumption instructions provided
    notes: "Implementation varies by tool (Claude native, others need alternatives)"

  complete:
    signature: "complete <project-name>"
    purpose: "Archive project and extract learnings"
    entry_point: false
    requires_context: true  # Must have active project
    outputs:
      - knowledge extraction analysis
      - agent files updated (with approval)
      - project archived to projects/completed/YYYY-MM/
      - GitHub issue closed (if linked)
      - worktree removed (if used)
      - patterns extracted to memory/recent/
```

#### Behavioral Contracts

Each command must guarantee:

**research**:
- ✅ Creates research project structure
- ✅ Enables deep analysis with specialist agents
- ✅ Outputs comprehensive findings document
- ✅ Does NOT require prior capture or issue
- ✅ Can lead to capture or start (but doesn't enforce)

**capture**:
- ✅ Creates GitHub issue immediately
- ✅ Auto-categorizes with appropriate labels
- ✅ Stores idea for later prioritization
- ✅ Optionally helps flesh out idea if user wants
- ✅ Does NOT automatically start project

**start**:
- ✅ Accepts issue# OR text (creates issue if text)
- ✅ Creates complete project structure
- ✅ Links to GitHub issue bidirectionally
- ✅ Sets up git workflow (branch, worktree if configured)
- ✅ Does NOT require prior capture or research

**switch**:
- ✅ Commits current work (WIP commit)
- ✅ Pushes to remote for backup
- ✅ Switches branch cleanly
- ✅ Preserves all work (non-destructive)
- ✅ Works with or without worktrees

**pause**:
- ✅ Saves conversation context
- ✅ Enables resumption in new session
- ✅ Preserves decisions, blockers, next steps
- ⚠️ Implementation tool-dependent

**complete**:
- ✅ Analyzes project for knowledge extraction
- ✅ Proposes updates (requires approval)
- ✅ Archives project with full history
- ✅ Closes linked GitHub issue
- ✅ Extracts patterns automatically
- ✅ Updates agent confidence scores

### Workflow State Machine

```
┌─────────────────────────────────────────────────────────┐
│ STATE: No Active Work                                   │
│                                                          │
│ Available Commands:                                      │
│   - research <text|issue#>  → STATE: Researching        │
│   - capture <text>          → STATE: No Active Work     │
│   - start <issue#|text>     → STATE: Active Development │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ STATE: Researching                                       │
│                                                          │
│ Available Commands:                                      │
│   - pause [desc]            → STATE: Paused             │
│   - complete <project>      → STATE: No Active Work     │
│   - capture <findings>      → STATE: No Active Work     │
│   - start <issue#|text>     → STATE: Active Development │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ STATE: Active Development                                │
│                                                          │
│ Available Commands:                                      │
│   - switch [branch]         → STATE: Active Development │
│   - pause [desc]            → STATE: Paused             │
│   - complete <project>      → STATE: No Active Work     │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ STATE: Paused                                            │
│                                                          │
│ Available Commands:                                      │
│   - resume                  → STATE: (previous state)   │
└─────────────────────────────────────────────────────────┘
```

**Key Property**: States are **loose** - commands are available based on context, not enforced gates.

---

## 2. Portable Agent Definition Format (v2)

### Problem with v1
Current agent definitions reference Claude Code-specific features:
- "Use Task tool to invoke specialist"
- "Delegate using subagent_type parameter"
- Tool-specific invocation instructions

### v2 Design: Tool-Invocation-Agnostic Format

```markdown
# [Agent Name] (Portable v2 Format)

## Role & Expertise
[Unchanged - tool-agnostic role description]

## Core Responsibilities
[Unchanged - tool-agnostic responsibilities]

## Capability Confidence Levels
[Unchanged - quantitative confidence scores]

### Primary Expertise (≥0.85)
[Unchanged - tasks agent handles directly]

### Secondary Expertise (0.60-0.84)
[Unchanged - tasks needing collaboration]

### Developing Areas (<0.60)
[Unchanged - tasks requiring delegation]

## Tools & Technologies Mastery
[Unchanged - tool knowledge, not invocation methods]

---
## TOOL-SPECIFIC INVOCATION (Append to each agent file)

### How to Use This Agent

**In Claude Code**:
\`\`\`
Task(subagent_type="agent-name", prompt="<task description>")
\`\`\`

**In Cursor**:
\`\`\`
@.claude/agents/.../agent-name.md
<Provide task description in chat>
\`\`\`

**In Windsurf**:
\`\`\`
# In Cascade flow:
- agent: agent-name
  persona_file: .claude/agents/.../agent-name.md
  task: "<task description>"
\`\`\`

**In Aider**:
\`\`\`bash
aider --read .claude/agents/.../agent-name.md
# Then provide task description
\`\`\`

**In Continue**:
\`\`\`
/agent agent-name
<Provide task description>
\`\`\`

---
## Delegation Decision Framework
[Unchanged - confidence-based delegation logic]

## Delegation Protocol (Tool-Agnostic)

**Step 1: Recognize need for specialist**
[Unchanged - confidence thresholds, quality criteria]

**Step 2: Prepare complete context**
[Unchanged - context requirements]

**Step 3: Invoke specialist (see Tool-Specific Invocation above)**
\`\`\`
DELEGATE TO: [specialist-name]
PROVIDE: [context package]
REQUEST: [expected deliverable]
\`\`\`

**Step 4: Validate specialist output**
[Unchanged - validation criteria]

**Step 5: Execute with confidence**
[Unchanged - implementation guidance]

---
[Rest of agent content unchanged]
```

### Migration Strategy

**Phase 1**: Add "Tool-Specific Invocation" section to all existing agents
**Phase 2**: Update delegation protocol references from "Task tool" → "see invocation above"
**Phase 3**: Validate all references are tool-agnostic

**Files to update** (14 agent files):
```
.claude/agents/roles/
  - analytics-engineer-role.md
  - data-engineer-role.md
  - data-architect-role.md
  - role-template.md

.claude/agents/specialists/
  - dbt-expert.md
  - snowflake-expert.md
  - tableau-expert.md
  - dlthub-expert.md
  - specialist-template.md
```

---

## 3. Tool Adapter Pattern Architecture

### Three-Layer Architecture

```
┌──────────────────────────────────────────────────────────────┐
│ LAYER 1: Tool-Specific Adapter (20% - varies by tool)       │
│                                                               │
│  Purpose: Translate tool-native invocation → UCI             │
│  Components:                                                  │
│    - Command registration (.cursorrules, flows, etc.)        │
│    - Agent invocation mechanisms                             │
│    - Task tracking integration                               │
│                                                               │
│  Example (Cursor):                                           │
│    .cursorrules → defines /start command                     │
│                → executes ./scripts/start.sh                 │
│                → formats output for Cursor                   │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│ LAYER 2: Portable Workflow Core (80% - unchanged)           │
│                                                               │
│  Purpose: Execute ADLC workflow logic                         │
│  Components:                                                  │
│    - Bash scripts (capture.sh, start.sh, etc.)              │
│    - GitHub integration (gh CLI)                             │
│    - Project structure management                            │
│    - Git workflow automation                                 │
│                                                               │
│  100% portable - works on any Unix system                    │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│ LAYER 3: Knowledge & Structure (100% - unchanged)           │
│                                                               │
│  Purpose: Store domain knowledge and project context          │
│  Components:                                                  │
│    - Agent definitions (.claude/agents/)                     │
│    - Memory system (.claude/memory/)                         │
│    - Project structure (projects/active/, completed/)        │
│                                                               │
│  Pure markdown files - universally readable                  │
└──────────────────────────────────────────────────────────────┘
```

### Adapter Responsibilities

**Each tool adapter must provide**:

1. **Command Registration**
   - Map UCI commands to tool-native invocation
   - Example: Claude `/start` → Cursor custom command → `./scripts/start.sh`

2. **Agent Coordination**
   - Implement one of the coordination patterns:
     - File-based handoff
     - Flow-based automation
     - Inline expertise loading
     - Manual context switching

3. **State Management**
   - Track active project (for switch/complete)
   - Preserve conversation context (for pause/resume)
   - Integrate with tool-native features where available

4. **Output Formatting**
   - Parse bash script output
   - Format for tool-native display
   - Provide next-step guidance

### Adapter Implementation Template

```yaml
# Tool Adapter Specification Template

tool_name: "[Cursor|Windsurf|Aider|Continue]"
version: "1.0"

commands:
  research:
    invocation_method: "[slash command|custom command|flow|CLI arg]"
    registration_file: "[.cursorrules|.windsurf/flows/research.yaml|alias]"
    script_execution: "./scripts/research.sh"
    output_format: "[tool-specific formatting]"

  capture:
    invocation_method: "[...]"
    registration_file: "[...]"
    script_execution: "./scripts/capture.sh"
    output_format: "[...]"

  # ... (repeat for all UCI commands)

agent_coordination:
  pattern: "[file-based|flow-based|inline|manual]"
  implementation:
    delegate: "[how to invoke specialist agent]"
    return: "[how specialist returns findings]"
    execute: "[how main agent gets findings]"

state_management:
  active_project_tracking: "[how to know current project]"
  context_preservation: "[how to save/resume context]"

integration_points:
  task_tracking: "[tool-native todo system OR custom]"
  git_workflow: "[tool git integration OR scripts]"
  multi_file_context: "[how tool handles multiple files]"
```

---

## 4. Agent Coordination Patterns (Without Task Tool)

Phase 1 identified **4 viable patterns**. Phase 2 specifies **implementation details** for each.

### Pattern 1: File-Based Handoff (Best for Cursor)

**Architecture**:
```
Main Agent (Composer)
    ↓ writes
tasks/specialist-request.md
    ↓ reads
Specialist Agent (@mention)
    ↓ writes
tasks/specialist-findings.md
    ↓ reads
Main Agent (Composer)
    ↓ executes
Recommendations
```

**Implementation Spec**:

```markdown
## File-Based Handoff Protocol v1.0

### Step 1: Main Agent Recognizes Delegation Need
**Condition**: Confidence <0.60 OR complex task requiring specialist

**Action**: Write delegation request file
\`\`\`markdown
<!-- tasks/dbt-expert-request.md -->
# Delegation Request: dbt-expert

**Requesting Agent**: analytics-engineer-role
**Date**: 2025-10-16
**Confidence**: 0.55 (below delegation threshold)

## Task
Optimize slow incremental model with complex deduplication logic

## Current State
- Model: customer_transactions
- Runtime: 2 hours (unacceptable)
- Row count: 50M
- Issue: Multiple updates per customer per day, deduplication needed

## Requirements
- Reduce runtime to <30 minutes
- Handle late-arriving data (3-day lookback)
- Maintain referential integrity with downstream marts

## Constraints
- Must deploy by Friday (3 days)
- Cannot change source schema
- Must preserve existing tests

## Expected Deliverable
- Optimized dbt model code with incremental config
- Performance analysis (before/after estimates)
- Test suite for validation
- Implementation instructions
\`\`\`

### Step 2: User Invokes Specialist (Cursor-specific)
**User action**: `@.claude/agents/specialists/dbt-expert.md analyze tasks/dbt-expert-request.md`

**Tool behavior**: Cursor Composer loads specialist context and request file

### Step 3: Specialist Analyzes and Writes Findings
**Specialist reads**: tasks/dbt-expert-request.md
**Specialist analyzes**: Using dbt/Snowflake expertise + MCP tools
**Specialist writes**: tasks/dbt-expert-findings.md

\`\`\`markdown
<!-- tasks/dbt-expert-findings.md -->
# Specialist Findings: dbt-expert

**Date**: 2025-10-16
**Request**: tasks/dbt-expert-request.md
**Status**: ✅ Complete

## Analysis Summary
Identified root cause: Missing unique_key config + no deduplication logic

## Recommended Solution

### Optimized Incremental Model
\`\`\`sql
{{
  config(
    materialized='incremental',
    unique_key='transaction_id',
    merge_update_columns=['amount', 'updated_at'],
    on_schema_change='fail'
  )
}}

WITH source_data AS (
  SELECT * FROM {{ ref('stg_customer_transactions') }}
  {% if is_incremental() %}
  WHERE transaction_date >= dateadd('day', -3, (SELECT MAX(transaction_date) FROM {{ this }}))
  {% endif %}
),

deduped AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY transaction_id
      ORDER BY updated_at DESC
    ) as rn
  FROM source_data
)

SELECT * EXCLUDE (rn)
FROM deduped
WHERE rn = 1
\`\`\`

### Performance Estimate
- **Current**: Full table scan, 2 hours
- **Optimized**: Incremental with 3-day lookback, ~15-20 minutes
- **Improvement**: 85-90% reduction

### Testing Strategy
\`\`\`yaml
tests:
  - dbt_utils.unique_combination_of_columns:
      combination_of_columns:
        - transaction_id
        - transaction_date
  - not_null: [transaction_id, customer_id, amount]
  - relationships:
      to: ref('customers')
      field: customer_id
\`\`\`

### Implementation Steps
1. Backup current model
2. Apply new incremental config
3. Run full-refresh: `dbt run --select customer_transactions --full-refresh`
4. Test incremental: `dbt run --select customer_transactions`
5. Validate row counts match expected
6. Deploy to production

### Validation Checklist
- [ ] Full refresh completes successfully
- [ ] Incremental run <30 minutes
- [ ] Row counts match source
- [ ] Downstream mart tests pass
- [ ] No duplicate transaction_ids
\`\`\`

### Step 4: Main Agent Reads Findings and Executes
**Main agent (Composer)**: Reads tasks/dbt-expert-findings.md
**Main agent validates**: Solution meets requirements
**Main agent executes**: Implements recommended solution
**Main agent confirms**: Tests pass, deployment successful

### Step 5: Cleanup
**After execution**: Archive or delete request/findings files
```

**Tool Support Requirements**:
- ✅ File creation/editing
- ✅ @ mention file loading
- ✅ Multi-file context awareness
- ✅ Markdown rendering

**Best For**: Cursor (excellent Composer support)

---

### Pattern 2: Flow-Based Automation (Best for Windsurf)

**Architecture**:
```
Cascade Flow Definition
    ↓
Main Agent Step
    ↓ (if delegation needed)
Specialist Agent Step (automatic)
    ↓
Main Agent Step (execution)
```

**Implementation Spec**:

```yaml
# analytics-workflow.yaml (Windsurf Cascade Flow)

name: "Analytics Engineering Workflow"
version: "1.0"
description: "ADLC workflow with automatic specialist delegation"

personas:
  analytics_engineer:
    file: ".claude/agents/roles/analytics-engineer-role.md"
    confidence_threshold: 0.60

  dbt_expert:
    file: ".claude/agents/specialists/dbt-expert.md"
    triggers:
      - confidence_below: 0.60
      - keywords: ["dbt macro", "incremental strategy", "complex transformation"]

  snowflake_expert:
    file: ".claude/agents/specialists/snowflake-expert.md"
    triggers:
      - confidence_below: 0.60
      - keywords: ["warehouse cost", "query optimization", "clustering"]

steps:
  - id: initial_analysis
    persona: analytics_engineer
    inputs:
      - user_request
    actions:
      - analyze_requirements
      - assess_confidence
      - determine_delegation_need
    outputs:
      - analysis_summary
      - delegation_decision

  - id: dbt_specialist_consultation
    persona: dbt_expert
    condition: "delegation_decision.specialist == 'dbt'"
    inputs:
      - analysis_summary
      - user_request
    actions:
      - deep_dbt_analysis
      - generate_recommendations
    outputs:
      - dbt_recommendations

  - id: snowflake_specialist_consultation
    persona: snowflake_expert
    condition: "delegation_decision.specialist == 'snowflake'"
    inputs:
      - analysis_summary
      - user_request
    actions:
      - warehouse_analysis
      - cost_optimization_recommendations
    outputs:
      - snowflake_recommendations

  - id: implementation
    persona: analytics_engineer
    inputs:
      - dbt_recommendations (if available)
      - snowflake_recommendations (if available)
    actions:
      - validate_recommendations
      - implement_solution
      - test_deployment
    outputs:
      - implementation_summary
      - test_results

transitions:
  - from: initial_analysis
    to: dbt_specialist_consultation
    condition: "delegation_decision.specialist == 'dbt'"

  - from: initial_analysis
    to: snowflake_specialist_consultation
    condition: "delegation_decision.specialist == 'snowflake'"

  - from: initial_analysis
    to: implementation
    condition: "delegation_decision.specialist == 'none'"

  - from: dbt_specialist_consultation
    to: implementation

  - from: snowflake_specialist_consultation
    to: implementation
```

**Invocation**:
```bash
# User triggers flow
windsurf run analytics-workflow --input "Optimize customer_transactions model"
```

**Tool Support Requirements**:
- ✅ Flow definition (YAML or similar)
- ✅ Persona switching
- ✅ Conditional branching
- ✅ Context passing between steps

**Best For**: Windsurf (native Cascade support)

---

### Pattern 3: Inline Expertise Loading (Universal Fallback)

**Architecture**:
```
Main Agent
    ↓ recognizes delegation need
Main Agent reads specialist file
    ↓ applies knowledge inline
Main Agent executes with specialist context
```

**Implementation Spec**:

```markdown
## Inline Expertise Protocol v1.0

### When to Use
- Tool lacks multi-agent coordination
- Quick consultation needed (not complex delegation)
- Single-context preference

### Process

**Step 1: Main Agent Recognizes Need**
"My confidence is 0.55 on dbt incremental strategies. I should consult dbt-expert knowledge."

**Step 2: Load Specialist Knowledge**
\`\`\`
[Main agent reads: .claude/agents/specialists/dbt-expert.md]
\`\`\`

**Step 3: Apply Inline**
"Based on dbt-expert patterns for incremental models with deduplication (confidence: 0.88):
- Use ROW_NUMBER() window function for deduplication
- 3-day lookback window for late-arriving data
- unique_key config on transaction_id
- merge_update_columns for changed fields"

**Step 4: Implement with Specialist Context**
[Implements solution using loaded knowledge]

**Step 5: Validate Against Specialist Criteria**
"Checking against dbt-expert validation checklist:
- ✅ Incremental config correct
- ✅ Deduplication logic sound
- ✅ Test coverage adequate"
```

**Pros**:
- ✅ Works in any tool (just read files)
- ✅ Simple, no coordination overhead
- ✅ Fast (no context switching)

**Cons**:
- ⚠️ Larger context window (specialist knowledge + current task)
- ⚠️ Less specialized (no dedicated specialist focus)
- ⚠️ Main agent must interpret specialist patterns

**Best For**: Aider, simple tasks, tools without multi-agent support

---

### Pattern 4: Manual Context Switching (Always Works)

**Architecture**:
```
Main Agent (User Session 1)
    ↓ identifies delegation need
User manually switches to specialist
    ↓
Specialist Agent (User Session 2)
    ↓ provides recommendations
User manually switches back
    ↓
Main Agent (User Session 1) executes
```

**Implementation Spec**:

```markdown
## Manual Context Switching Protocol v1.0

### Step 1: Main Agent Signals Delegation Need
**Main agent**: "This task requires dbt-expert consultation (confidence: 0.55).

Please:
1. Copy this context:
   - Task: Optimize customer_transactions incremental model
   - Current runtime: 2 hours
   - Target: <30 minutes
   - Constraints: 3-day lookback, preserve tests

2. Switch to dbt-expert:
   [Tool-specific instruction]

3. Get recommendations

4. Return to this session with findings"

### Step 2: User Switches to Specialist

**In Claude Code**:
\`\`\`
Task(subagent_type="dbt-expert", prompt="[copied context]")
\`\`\`

**In Aider**:
\`\`\`bash
aider --read .claude/agents/specialists/dbt-expert.md
# Paste context
\`\`\`

**In Cursor**:
\`\`\`
New chat: @.claude/agents/specialists/dbt-expert.md
# Paste context
\`\`\`

**In Continue**:
\`\`\`
/agent dbt-expert
# Paste context
\`\`\`

### Step 3: Specialist Provides Recommendations
[Specialist analyzes and provides detailed recommendations]

### Step 4: User Returns with Findings
**User copies recommendations back to main session**

### Step 5: Main Agent Validates and Executes
**Main agent**: "Reviewing dbt-expert recommendations...
- Incremental config looks correct
- Deduplication logic is sound
- Performance estimate is reasonable
- Tests are comprehensive

Proceeding with implementation..."
```

**Pros**:
- ✅ Works in ALL tools
- ✅ User has full control
- ✅ No tool feature dependencies

**Cons**:
- ⚠️ Fully manual (most labor intensive)
- ⚠️ Context copying required
- ⚠️ No automation

**Best For**: Fallback option, tools with limited features, complex scenarios requiring human oversight

---

## 5. Tool-Specific Design Specifications

### Cursor Adapter Design

**Command Registration** (`.cursorrules`):
```markdown
# DA Agent Hub - Cursor Adapter
# Version: 1.0

## Custom Commands

### /research
When user types "/research <text|issue#>":
1. Execute: `./scripts/research.sh "<input>"`
2. Display output with formatting
3. Suggest next steps: /capture or /start

### /capture
When user types "/capture <idea>":
1. Execute: `./scripts/capture.sh "<idea>"`
2. Display GitHub issue URL
3. Suggest: View in GitHub, /research for deeper analysis, /start when ready

### /start
When user types "/start <issue#|text>":
1. Execute: `./scripts/start.sh "<input>"`
2. Display project location and structure
3. Load project files in Composer context
4. Suggest: Review spec.md, begin implementation

### /switch
When user types "/switch [branch]":
1. Execute: `./scripts/switch.sh "[branch]"`
2. Display current branch after switch
3. Suggest next actions

### /complete
When user types "/complete <project>":
1. Execute: `./scripts/finish.sh "<project>"`
2. Analyze project files for knowledge extraction
3. Propose updates to agent files/docs
4. Get user approval
5. Apply approved updates
6. Display archive location

## Agent Coordination (File-Based)

When main agent needs specialist consultation:
1. Write: `tasks/[specialist]-request.md`
2. Instruct user: "@.claude/agents/specialists/[specialist].md analyze tasks/[specialist]-request.md"
3. Specialist writes: `tasks/[specialist]-findings.md`
4. Main agent reads findings
5. Execute recommendations

## Project Context Management

On /start:
- Add to Composer context: spec.md, context.md, README.md
- Monitor: tasks/ directory for findings

During work:
- Keep spec.md, context.md in context
- Update context.md with progress

On /complete:
- Read all project files for analysis
- Extract patterns from tasks/*-findings.md
```

**Tool Capabilities Leveraged**:
- ✅ Composer for multi-file editing
- ✅ @ mentions for agent invocation
- ✅ Custom commands via .cursorrules
- ✅ Excellent file context awareness

**Coordination Pattern**: File-Based Handoff

---

### Windsurf Adapter Design

**Flow Definitions** (`.windsurf/flows/`):

```yaml
# .windsurf/flows/adlc-workflow.yaml

name: "DA Agent Hub ADLC Workflow"
version: "1.0"

commands:
  research:
    trigger: "/research"
    flow: research_flow

  capture:
    trigger: "/capture"
    flow: capture_flow

  start:
    trigger: "/start"
    flow: start_flow

  complete:
    trigger: "/complete"
    flow: complete_flow

personas:
  analytics_engineer:
    file: ".claude/agents/roles/analytics-engineer-role.md"
  data_engineer:
    file: ".claude/agents/roles/data-engineer-role.md"
  dbt_expert:
    file: ".claude/agents/specialists/dbt-expert.md"
  snowflake_expert:
    file: ".claude/agents/specialists/snowflake-expert.md"

flows:
  research_flow:
    steps:
      - shell: "./scripts/research.sh {input}"
      - display_output
      - suggest_next: ["capture", "start"]

  capture_flow:
    steps:
      - shell: "./scripts/capture.sh {input}"
      - display_github_issue
      - suggest_next: ["research", "start", "github_view"]

  start_flow:
    steps:
      - shell: "./scripts/start.sh {input}"
      - load_project_context:
          files: ["spec.md", "context.md", "README.md"]
      - suggest_next: ["review_spec", "begin_work"]

  complete_flow:
    steps:
      - analyze_project:
          persona: analytics_engineer
          files: ["spec.md", "context.md", "tasks/*"]
      - propose_knowledge_updates
      - wait_approval
      - apply_updates:
          condition: approved
      - shell: "./scripts/finish.sh {project_name}"
      - display_completion_summary

knowledge_base:
  import_paths:
    - ".claude/memory/patterns/**/*.md"
    - ".claude/agents/**/*.md"
  index_for_search: true
```

**Tool Capabilities Leveraged**:
- ✅ Cascade flows for automation
- ✅ Persona switching
- ✅ Knowledge base integration
- ✅ Conditional logic in flows

**Coordination Pattern**: Flow-Based Automation

---

### Aider Adapter Design

**Wrapper Scripts** (`~/bin/aider-da-hub/`):

```bash
#!/bin/bash
# aider-research - DA Agent Hub research command

INPUT="$1"

echo "🔬 Starting research mode..."
./scripts/research.sh "$INPUT"

echo ""
echo "📚 Research project created. To work on it with Aider:"
echo "  aider --read projects/active/research-*/spec.md --read .claude/agents/roles/analytics-engineer-role.md"
```

```bash
#!/bin/bash
# aider-start - DA Agent Hub start command

INPUT="$1"

echo "🚀 Starting project..."
./scripts/start.sh "$INPUT"

# Extract project directory from output
PROJECT_DIR=$(ls -td projects/active/* | head -1)

echo ""
echo "✅ Project created: $PROJECT_DIR"
echo ""
echo "To begin work with Aider:"
echo "  aider --read $PROJECT_DIR/spec.md --read $PROJECT_DIR/context.md [files-to-edit]"
echo ""
echo "To consult specialist:"
echo "  aider --read .claude/agents/specialists/dbt-expert.md"
```

**Shell Aliases** (`.bashrc` or `.zshrc`):
```bash
# DA Agent Hub Commands
alias da-research='~/bin/aider-da-hub/aider-research'
alias da-capture='./scripts/capture.sh'
alias da-start='~/bin/aider-da-hub/aider-start'
alias da-switch='./scripts/switch.sh'
alias da-complete='./scripts/finish.sh'

# Aider with DA Agent Hub context
alias aider-analytics='aider --read .claude/agents/roles/analytics-engineer-role.md'
alias aider-dbt='aider --read .claude/agents/specialists/dbt-expert.md'
alias aider-snowflake='aider --read .claude/agents/specialists/snowflake-expert.md'
```

**Tool Capabilities Leveraged**:
- ✅ CLI-native (perfect for shell scripts)
- ✅ --read flag for context loading
- ✅ Git-centric workflow
- ✅ Direct file editing

**Coordination Pattern**: Manual Context Switching + Inline Expertise

---

## 6. Project Structure Adaptations

### Universal Structure (Unchanged)

```
projects/
├── active/
│   └── feature-project-name/
│       ├── README.md           # Navigation hub
│       ├── spec.md            # Stable requirements
│       ├── context.md         # Dynamic state
│       └── tasks/             # Agent coordination
│           ├── current-task.md
│           └── [agent]-findings.md
└── completed/
    └── YYYY-MM/
        └── feature-project-name/
            └── [same structure]
```

### Tool-Specific Enhancements

#### Cursor Enhancements
```
feature-project-name/
├── .cursor/                    # Cursor-specific
│   └── context.json           # Composer preferred files
└── [standard structure]
```

**`.cursor/context.json`**:
```json
{
  "preferredFiles": [
    "spec.md",
    "context.md",
    "tasks/current-task.md"
  ],
  "excludedDirectories": []
}
```

#### Windsurf Enhancements
```
feature-project-name/
├── .windsurf/                  # Windsurf-specific
│   ├── project-flow.yaml      # Project-specific flow
│   └── personas.yaml          # Active agent personas
└── [standard structure]
```

**`.windsurf/project-flow.yaml`**:
```yaml
project: "feature-project-name"
default_persona: "analytics_engineer"
context_files:
  - "spec.md"
  - "context.md"
  - "tasks/*.md"
```

#### Aider Enhancements
```
feature-project-name/
├── .aiderignore               # Aider-specific ignore
├── AIDER-CONTEXT.md           # Quick reference for --read
└── [standard structure]
```

**`AIDER-CONTEXT.md`**:
```markdown
# Aider Context Quick Reference

## Start Editing
\`\`\`bash
aider --read spec.md --read context.md [files-to-edit]
\`\`\`

## Consult Specialists
\`\`\`bash
# dbt expertise
aider --read ../.claude/agents/specialists/dbt-expert.md

# Snowflake expertise
aider --read ../.claude/agents/specialists/snowflake-expert.md
\`\`\`

## Project Files
- **spec.md**: Stable requirements
- **context.md**: Current state, blockers, next steps
- **tasks/**: Agent coordination files
```

### Optional Enhancements (All Tools)

**`.da-hub/config.json`** (Project-level config):
```json
{
  "project_name": "feature-customer-analytics",
  "github_issue": 123,
  "worktree_enabled": true,
  "primary_agent": "analytics-engineer-role",
  "specialists_used": ["dbt-expert", "snowflake-expert"],
  "created": "2025-10-16",
  "last_modified": "2025-10-16"
}
```

**Benefits**:
- Programmatic access to project metadata
- Tool adapters can read config
- Enables automation and tooling

---

## 7. Phase 2 Summary & Next Steps

### Deliverables Completed

✅ **Universal Command Interface (UCI)**:
- Standardized command signatures
- Behavioral contracts
- State machine definition

✅ **Portable Agent Format (v2)**:
- Tool-invocation-agnostic structure
- Migration strategy for 14 agent files
- Tool-specific invocation section template

✅ **Adapter Pattern Architecture**:
- Three-layer design (20% adapter, 80% portable)
- Adapter responsibility specification
- Implementation template

✅ **Agent Coordination Patterns**:
- 4 patterns with detailed implementation specs:
  1. File-Based Handoff (Cursor)
  2. Flow-Based Automation (Windsurf)
  3. Inline Expertise Loading (Universal)
  4. Manual Context Switching (Fallback)

✅ **Tool-Specific Designs**:
- Cursor adapter (.cursorrules, file-based coordination)
- Windsurf adapter (flows, personas, knowledge base)
- Aider adapter (shell scripts, aliases, manual coordination)

✅ **Project Structure Adaptations**:
- Universal structure (unchanged)
- Tool-specific enhancements (optional)
- Metadata config (for automation)

### Key Design Decisions

1. **80/20 Architecture**: 80% shared portable core, 20% tool adapter
2. **Non-Linear Workflow**: Commands work independently, no enforced sequence
3. **Multiple Coordination Patterns**: Support different tool capabilities
4. **Backward Compatibility**: v2 agent format enhances, doesn't break v1
5. **Optional Enhancements**: Tools can add features without breaking portability

### Phase 3 Preview: Reference Implementations

**Next Tasks**:
1. Implement Cursor adapter
   - Create .cursorrules file
   - Test file-based coordination
   - Validate full ADLC workflow

2. Implement Windsurf adapter
   - Create Cascade flows
   - Define personas
   - Test flow-based automation

3. Validation testing
   - Same project in both tools
   - Compare outcomes
   - Measure portability score

4. Documentation
   - Setup guides (Cursor, Windsurf)
   - Migration guide from Claude Code
   - Adaptation guide for new tools

**Estimated Effort**: 4-5 days
**Success Criteria**: ≥80% workflow features working in both tools

---

*Phase 2 abstraction design complete. Ready for stakeholder review and Phase 3 implementation.*
