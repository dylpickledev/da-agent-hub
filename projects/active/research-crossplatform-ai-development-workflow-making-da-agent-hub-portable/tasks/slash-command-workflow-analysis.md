# Slash Command Workflow Analysis

**Project**: Cross-Platform AI Development Workflow
**Purpose**: Document the actual (non-linear) workflow patterns enabled by slash commands
**Date**: 2025-10-16

---

## Executive Summary

The DA Agent Hub slash commands **don't enforce a linear workflow**. Instead, they provide **entry points and transitions** that support multiple paths based on user context and needs.

**Key Insight**: Users enter the workflow at different points depending on their starting state:
- **Fuzzy idea** → `/research` (explore and define)
- **Clear idea** → `/capture` (store and optionally flesh out)
- **Ready to build** → `/start` (from issue or ad-hoc)
- **Context switch needed** → `/switch` (preserve work, change focus)
- **Temporary pause** → `/pause` (save conversation state)
- **Project complete** → `/complete` (archive and learn)

---

## Non-Linear Workflow Map

```
                    ┌──────────────────────────────────────┐
                    │  Entry Points (Context-Dependent)   │
                    └──────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐          ┌───────────────┐          ┌───────────────┐
│  /research    │          │   /capture    │          │    /start     │
│               │          │               │          │               │
│ "I have a     │          │ "I have a     │          │ "I'm ready    │
│ fuzzy idea,   │          │ clear idea,   │          │ to build      │
│ help me       │          │ capture it    │          │ this now"     │
│ explore it"   │          │ for later"    │          │               │
└───────┬───────┘          └───────┬───────┘          └───────┬───────┘
        │                          │                          │
        │ Can lead to →            │ Can lead to →            │
        ├─────────────────────────→│                          │
        │                          ├─────────────────────────→│
        │                          │                          │
        └──────────────────────────┴──────────────────────────┘
                                   │
                                   ▼
                        ┌───────────────────┐
                        │  Active Work      │
                        │  (Development)    │
                        └─────────┬─────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌───────────────┐        ┌───────────────┐        ┌───────────────┐
│   /switch     │        │    /pause     │        │   /complete   │
│               │        │               │        │               │
│ "Need to      │        │ "Save my      │        │ "Finished,    │
│ work on       │        │ progress,     │        │ archive and   │
│ something     │        │ resume        │        │ extract       │
│ else"         │        │ later"        │        │ learnings"    │
└───────┬───────┘        └───────────────┘        └───────┬───────┘
        │                                                  │
        │                                                  │
        └──────────────────────────────────────────────────┘
                          Can start new work
```

---

## Command Relationships & Use Cases

### Entry Point Commands (Where You Start)

#### `/research [text|issue#]` - Exploratory Start
**When to use**:
- ❓ "I have a vague idea but need to understand feasibility"
- ❓ "Should we even do this? What are the implications?"
- ❓ "What would this involve? I need a deeper analysis"

**What it does**:
1. Creates research project structure
2. Enables deep exploration with specialist agents
3. Analyzes technical approach, feasibility, dependencies
4. Outputs comprehensive analysis document

**Leads to**:
- `/capture` - If research confirms idea is good, capture for roadmap
- `/start` - If research shows clear path and ready to build immediately
- Abandon - If research reveals blockers or low ROI

**Example flow**:
```
User: "Not sure if we should build real-time analytics dashboard"
  ↓
/research "real-time analytics dashboard"
  ↓
Deep analysis: feasibility, architecture, complexity, alternatives
  ↓
Decision: "Yes, let's do it" → /capture (or /start immediately)
Decision: "No, too complex" → Abandon, document findings
```

#### `/capture "idea"` - Clear Idea Storage
**When to use**:
- 💡 "I have a clear idea I want to remember"
- 💡 "Let's get this into the backlog for prioritization"
- 💡 "Brainstorming session - capture multiple ideas quickly"

**What it does**:
1. Creates GitHub issue with idea label
2. Auto-categorizes (bi-analytics, data-engineering, etc.)
3. Stores for later prioritization
4. **Optionally**: Can help flesh out idea immediately if user wants

**Leads to**:
- GitHub issue management (prioritization, roadmap planning)
- `/research [issue#]` - If need deeper analysis before building
- `/start [issue#]` - When ready to build

**Example flow**:
```
User: "Create customer churn prediction model"
  ↓
/capture "Create customer churn prediction model"
  ↓
Issue #123 created with 'idea' and 'analytics-engineering' labels
  ↓
Later... when prioritized:
  ↓
/start 123
```

**Optional immediate expansion**:
```
User: "Create customer churn prediction model, but help me think through requirements"
  ↓
/capture "Create customer churn prediction model"
  ↓
Issue created
  ↓
Claude: "Let's flesh this out. What defines churn? What data sources?..."
  ↓
Enhanced issue description
  ↓
Ready for /start when prioritized
```

#### `/start [issue#|"text"]` - Build Start (Flexible)
**When to use**:
- ✅ "I'm ready to build this specific issue"
- ✅ "I need to start building NOW, create issue on-the-fly"
- ✅ "Let's implement this idea we've been discussing"

**What it does**:
- **From issue**: Fetches existing issue, creates project structure
- **From text**: Creates issue first, then project structure
- Sets up complete development environment
- Creates git worktree (if configured)
- Links project ↔ GitHub issue

**Leads to**:
- Active development work
- `/switch` - If need to context switch
- `/pause` - If need to save progress
- `/complete` - When finished

**Example flows**:

**Flow A: Start from existing issue**
```
/start 85
  ↓
Fetches issue #85 details
  ↓
Creates: projects/active/feature-executive-kpi-dashboard/
  ↓
Development begins
```

**Flow B: Start from scratch (ad-hoc)**
```
/start "Fix broken dashboard filters"
  ↓
Creates issue #150
  ↓
Creates: projects/active/fix-broken-dashboard-filters/
  ↓
Development begins
```

**Flow C: Start after research**
```
/research "Snowflake cost optimization"
  ↓
Analysis complete, approach clear
  ↓
/start "Implement Snowflake cost optimization strategies"
  ↓
Development begins (research context preserved)
```

---

### Mid-Work Commands (While Building)

#### `/switch [optional-branch]` - Context Switching
**When to use**:
- 🔄 "Urgent issue came up, need to switch projects"
- 🔄 "Done with current work, move to next priority"
- 🔄 "Need to review something else without losing current context"

**What it does**:
1. Commits current work (WIP commit)
2. Pushes to remote (backup)
3. Switches to specified branch OR main
4. Clean working directory for new work

**Leads to**:
- New `/start` for different project
- Work on different existing branch
- Return to original work later (branch preserved)

**Example flows**:

**Flow A: Emergency context switch**
```
Working on: feature-customer-dashboard
  ↓
Manager: "Production bug! Fix ASAP"
  ↓
/switch
  ↓
Current work saved and pushed
  ↓
Switched to main, clean workspace
  ↓
/start "Fix production data pipeline bug"
  ↓
Fix applied, deployed
  ↓
/complete
  ↓
/switch feature-customer-dashboard
  ↓
Back to original work
```

**Flow B: Planned project rotation**
```
Finishing morning session on Project A
  ↓
/switch
  ↓
Project A saved
  ↓
/start [afternoon-project-issue]
  ↓
Work continues on Project B
```

#### `/pause [description]` - Save Conversation State
**When to use**:
- 💾 "I need to stop but want to resume exactly where I left off"
- 💾 "End of day, want to continue tomorrow"
- 💾 "Complex problem, need to step away and come back"

**What it does**:
1. Saves current conversation context
2. Captures current state, decisions, blockers
3. Enables seamless resumption in new session
4. **Note**: Claude-native feature, no script

**Leads to**:
- New Claude session later
- Conversation resume with full context

**Example flow**:
```
Mid-implementation of complex feature
  ↓
/pause "Implementing incremental model deduplication logic,
        deciding between ROW_NUMBER vs QUALIFY approach"
  ↓
Context saved
  ↓
[Next day, new Claude session]
  ↓
Claude resumes with full context of decision point
```

---

### Completion Command (Finish & Learn)

#### `/complete [project-name]` - Project Completion & Knowledge Extraction
**When to use**:
- ✅ "Project is done, archive it and learn from it"
- ✅ "Feature deployed, extract patterns for future work"
- ✅ "Finished implementation, update agent knowledge"

**What it does**:
1. **Analyzes project** for extractable knowledge
2. **Proposes updates** to agent files, documentation, memory
3. **Gets approval** before making changes
4. **Executes approved updates**
5. **Archives project** to completed/
6. **Closes GitHub issue** (if linked)
7. **Cleans up worktree** (if used)
8. **Extracts patterns** to memory system

**Leads to**:
- Enhanced agent capabilities (confidence scores updated)
- New patterns in knowledge base
- Improved future project execution
- New `/start` for next project

**Example flow**:
```
Feature development complete
  ↓
/complete feature-customer-churn-prediction
  ↓
Claude analyzes project findings
  ↓
Claude proposes:
  - Update analytics-engineer-role.md (new churn model pattern)
  - Add to knowledge/da-agent-hub/ml-patterns.md
  - Update dbt-expert confidence (+0.10 for ML models)
  ↓
User approves
  ↓
Knowledge updates applied
  ↓
Project archived to: projects/completed/2025-10/
  ↓
Issue #123 closed with completion comment
  ↓
Patterns extracted to: .claude/memory/recent/2025-10.md
  ↓
Ready for next project
```

---

## Real-World Workflow Scenarios

### Scenario 1: Brainstorming to Execution
**Context**: Weekly planning meeting, capturing multiple ideas

```
[During meeting - rapid capture]
/capture "Real-time safety incident dashboard"
/capture "Automated cost anomaly detection"
/capture "Customer analytics data mart"

[Later - prioritization in GitHub]
Review issues, add milestones, sort by priority

[Ready to build top priority]
/start 87  (issue #87: Real-time safety incident dashboard)
  ↓
Development work
  ↓
/complete feature-real-time-safety-incident-dashboard
```

### Scenario 2: Uncertain Idea to Validated Project
**Context**: Stakeholder request with unclear scope

```
Stakeholder: "We need better Snowflake cost management"

/research "Snowflake cost optimization strategies"
  ↓
Deep analysis:
  - Current spend patterns
  - Warehouse utilization
  - Query optimization opportunities
  - Clustering strategies
  ↓
Research output: "3 high-impact optimizations identified"
  ↓
/capture "Implement Snowflake cost optimization: clustering + warehouse sizing + query refactoring"
  ↓
Issue #99 created with detailed requirements from research
  ↓
[Later, when prioritized]
/start 99
  ↓
Implementation with research context
  ↓
/complete
```

### Scenario 3: Interrupted Work with Context Switch
**Context**: Working on feature when urgent bug reported

```
[Morning - Feature work]
/start 78  (New BI metrics dashboard)
  ↓
Implementing dashboard, halfway done
  ↓
[Urgent production bug reported]
  ↓
/switch
  ↓
Work saved, pushed to remote
  ↓
/start "Fix critical data pipeline failure in production"
  ↓
Bug fixed and deployed
  ↓
/complete fix-critical-data-pipeline-failure
  ↓
[Return to original work]
/switch feature-new-bi-metrics-dashboard
  ↓
Resume exactly where left off
  ↓
[End of day]
/pause "Dashboard layout complete, next: implement filters and date range selector"
  ↓
[Next morning]
Continue with full context from pause
```

### Scenario 4: Ad-Hoc Quick Fix (No Prior Issue)
**Context**: Small bug found during testing, fix immediately

```
[No existing issue]
/start "Fix incorrect date formatting in customer report"
  ↓
Issue auto-created (#145)
  ↓
Project structure created
  ↓
Quick fix implemented
  ↓
/complete fix-incorrect-date-formatting-in-customer-report
  ↓
Issue #145 automatically closed
  ↓
Pattern extracted if relevant
```

### Scenario 5: Multi-Day Research Project
**Context**: Large architectural decision needs deep analysis

```
/research "Should we migrate from Prefect to Orchestra for pipeline orchestration?"
  ↓
Research project created
  ↓
[Day 1] Analyze current Prefect usage
  ↓
/pause "Completed Prefect analysis, next: deep-dive Orchestra capabilities"
  ↓
[Day 2] Research Orchestra features, costs, migration path
  ↓
/pause "Orchestra research complete, next: compare trade-offs and build decision matrix"
  ↓
[Day 3] Create comparison matrix, recommendations
  ↓
Research complete with comprehensive findings
  ↓
Decision: "Yes, migrate" → /capture "Migrate pipeline orchestration from Prefect to Orchestra"
Decision: "No, stay with Prefect" → Document findings, no migration
```

---

## Command Dependencies & Relationships

### Independent Entry Points (No Prerequisites)
- ✅ `/research` - Can start anytime with text or issue
- ✅ `/capture` - Can start anytime with idea text
- ✅ `/start` - Can start with issue# or text

### Requires Active Project Context
- ⚠️ `/switch` - Assumes current work in progress
- ⚠️ `/pause` - Assumes active conversation
- ⚠️ `/complete` - Requires project in projects/active/

### Can Lead To (Transitions)
```
/research  →  /capture (idea validated)
           →  /start   (ready to build immediately)
           →  Abandon  (not viable)

/capture   →  GitHub issue management
           →  /research (need deeper analysis)
           →  /start   (when prioritized)

/start     →  Development work
           →  /switch  (context change needed)
           →  /pause   (save progress)
           →  /complete (finished)

/switch    →  /start   (new project)
           →  Return   (back to switched-from work)

/pause     →  Resume   (new session, same context)

/complete  →  /start   (next project)
           →  Knowledge base updated for future work
```

---

## Key Differences from Linear Workflows

### What This Is NOT:
❌ **Not a strict sequence**: capture → research → start → complete
❌ **Not required progression**: Can skip steps based on context
❌ **Not enforced gates**: No "must research before start" requirement

### What This IS:
✅ **Flexible entry points**: Start where you are (fuzzy idea, clear idea, ready to build)
✅ **Context-aware transitions**: Move between commands based on needs
✅ **Non-destructive workflow**: Can pause, switch, return without losing work
✅ **Adaptive learning**: /complete improves future workflows regardless of entry point

---

## Portability Implications for Cross-Platform Design

### Critical Insights for Phase 2:

1. **Commands are not a pipeline**
   - Each command must work independently
   - Cross-command references are optional, not required
   - Tools need flexibility, not sequence enforcement

2. **Entry points matter more than flow**
   - Users must be able to start anywhere
   - No "you must research before start" logic
   - Adapt to user's current context

3. **State preservation is key**
   - /switch and /pause enable workflow flexibility
   - Tools need: git state management, context saving
   - Non-linear workflows require robust state handling

4. **Completion triggers learning**
   - /complete is the knowledge extraction point
   - Happens regardless of entry path (research→start→complete OR start→complete)
   - Must work with ad-hoc projects (no prior issue)

### Design Principles for Tool Adapters:

**Must Support**:
- ✅ Multiple entry points (research, capture, start)
- ✅ Ad-hoc project creation (/start "text")
- ✅ Context switching (/switch)
- ✅ State preservation (/pause)
- ✅ Knowledge extraction (/complete)

**Nice to Have**:
- ⚠️ Suggested transitions (after /research, offer /capture or /start)
- ⚠️ Workflow visualization (show where you are)
- ⚠️ Command discoverability (what can I do now?)

**Don't Enforce**:
- ❌ Linear progression
- ❌ Required steps
- ❌ Strict gates

---

## Recommended Documentation Updates

### For Phase 2 (Abstraction Design):

1. **Update spec.md**: Reflect non-linear workflow in implementation plan
2. **Create workflow decision tree**: Help users choose entry point
3. **Document command relationships**: Show flexible transitions, not sequences
4. **Tool adapter requirements**: Each tool must support all entry points independently

### For CLAUDE.md / README.md:

```markdown
## DA Agent Hub Workflow (Non-Linear)

The workflow is **flexible** - start where you are:

### Entry Points
- **Fuzzy idea?** → `/research` (explore feasibility)
- **Clear idea?** → `/capture` (store for later)
- **Ready to build?** → `/start` (from issue or ad-hoc)

### During Work
- **Switch projects?** → `/switch` (preserve work)
- **Save progress?** → `/pause` (resume later)

### Completion
- **Finished?** → `/complete` (archive + learn)

### No Required Sequence
You can:
- Go straight to `/start` without `/capture` or `/research`
- Do `/research` without ever `/start`ing
- `/capture` many ideas without building any
- `/start` ad-hoc work without GitHub issues
```

---

## Conclusion

The DA Agent Hub slash commands support a **non-linear, context-aware workflow** where:

1. **Entry points** depend on clarity level (fuzzy → research, clear → capture, ready → start)
2. **Transitions** are flexible, not enforced (can skip steps)
3. **Mid-work commands** enable interruption and resumption (/switch, /pause)
4. **Completion** triggers learning regardless of entry path (/complete)

**For portability**: Tool adapters must support **independent command invocation**, not sequential pipelines.

---

*This analysis will inform Phase 2 abstraction design to ensure cross-platform implementations maintain workflow flexibility.*
