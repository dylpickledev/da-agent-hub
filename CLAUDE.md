don't look at the full .env file. Only search for the var names up to the equals sign.

# DA Agent Hub: Analytics Development Lifecycle (ADLC) AI Platform

## Quick Start (First Time Setup)

### 1. Configure MCP for dbt Cloud Access

**IMPORTANT**: MCP (Model Context Protocol) integration with dbt Cloud is required for DA Agent Hub to work.

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your credentials
# - Get API token: https://cloud.getdbt.com/settings/tokens
# - Get Account ID from URL: https://cloud.getdbt.com/accounts/<ID>

# Validate configuration
./scripts/validate-mcp.sh
```

### 2. Restart Claude Code (REQUIRED)

**CRITICAL**: MCP servers only load when Claude Code starts.

1. Exit Claude Code completely (Cmd+Q on Mac, or kill the process)
2. Restart Claude Code in this directory
3. Verify MCP is loaded

### 3. Verify MCP is Working

```bash
# Check MCP servers loaded
claude /mcp
# Should show: dbt - dbt Cloud integration

# Test it
claude "List my dbt Cloud jobs"
```

### 4. Run Setup Wizard (Optional)

```bash
claude /setup
# Customizes DA Agent Hub for your specific data stack and role
```

**Troubleshooting**: If you encounter issues, see `docs/troubleshooting-mcp.md` or ask the onboarding-agent:
```bash
claude "I need help with DA Agent Hub setup" --agent onboarding-agent
```

---

## Quick Reference

**Security**: See `.claude/memory/patterns/git-workflow-patterns.md` for protected branch rules
**Testing**: See `.claude/memory/patterns/testing-patterns.md` for ADLC testing framework
**Cross-System Analysis**: See `.claude/memory/patterns/cross-system-analysis-patterns.md` for multi-tool coordination

### Mandatory Workflow (CRITICAL - Never Violate)
1. **ALWAYS create feature branch** before making any code changes
2. **ALWAYS create Pull Request** for code review and approval
3. **NEVER push directly** to protected branches
4. **NEVER merge without approval** (except for da-agent-hub documentation updates)

## Three-Layer Architecture

```
💡 LAYER 1: PLAN → Ideation & strategic planning with AI organization
🔧 LAYER 2: DEVELOP + TEST + DEPLOY → Local development with specialist agents
🤖 LAYER 3: OPERATE + OBSERVE + DISCOVER + ANALYZE → Automated operations
```

## Simplified Workflow Commands

### Essential Commands (Use Slash Commands)
1. **`/capture "[idea]"`** → Quick idea capture (creates GitHub issues)
2. **`/research [text|issue#]`** → Deep exploration and analysis (pre-capture or issue analysis)
3. **`/start [issue#|"text"]`** → Begin development (from issue OR creates issue from text + starts)
4. **`/switch [optional-branch]`** → Zero-loss context switching with automated backup
5. **`/complete [project]`** → Complete and archive projects (closes GitHub issue + cleans up worktree)

### Support Commands
6. **`/pause [description]`** → Save conversation context for seamless resumption
7. **`/setup-worktrees`** → One-time VS Code worktree integration setup

**Note**: For roadmap planning and prioritization, use GitHub's native issue management (labels, milestones, projects).

### Deprecated (Still Work, But Use New Names)
- **`/build`** → Use `/start` instead

### Underlying Scripts (Called by Slash Commands)
- `/capture` → `./scripts/capture.sh`
- `/research` → `./scripts/research.sh`
- `/start` → `./scripts/start.sh`
- `/switch` → `./scripts/switch.sh`
- `/complete` → `./scripts/finish.sh`
- `/pause` → (Claude-native, no script)
- `/setup-worktrees` → `./scripts/setup-worktrees.sh`

**Note**: Prefer slash commands for better Claude integration. Scripts can be run directly if needed.

### Repository Management
- **`./scripts/pull-all-repos.sh`** → Pull latest from all repos defined in `config/repositories.json`
  - Updates all knowledge repos (da_obsidian, da_team_documentation)
  - Updates all data stack repos (orchestration, ingestion, transformation, front_end, operations)
  - Uses correct branch for each repo (main, master, dbt_dw, etc.)
  - Organized output by category with color coding

### GitHub Issues Integration
All ideas are managed as GitHub issues with 'idea' label:
- **View all ideas**: `gh issue list --label idea --state open`
- **Filter by category**: `gh issue list --label idea --label bi-analytics`
- **Search ideas**: `gh issue list --label idea --search "dashboard"`
- **Track progress**: Issues automatically labeled 'in-progress' when built

## Project File Structure

```
projects/active/<project-name>/
├── README.md           # Navigation hub with quick links and progress
├── spec.md            # Project specification (stable requirements)
├── context.md         # Working context (dynamic state tracking)
└── tasks/             # Agent coordination directory
    ├── current-task.md     # Current agent assignments
    └── <tool>-findings.md  # Detailed agent findings
```

**File Purposes**:
- **README.md**: Entry point for navigation, progress summary, key decisions
- **spec.md**: Stable project requirements, end goals, implementation plan, success criteria
- **context.md**: Dynamic state tracking - branches, PRs, blockers, current focus
- **tasks/**: Agent coordination - task assignments and detailed findings

## Role-Based Agent System

### Agent Creation Guidelines

**ALWAYS use templates when creating new agents**:
- **New role agent**: Copy `.claude/agents/roles/role-template.md`
- **New specialist agent**: Copy `.claude/agents/specialists/specialist-template.md`

Templates encode correct architecture patterns:
- Roles delegate to specialists (80% independent, 20% consultation)
- Specialists use MCP tools + expertise (correctness-first)
- Both include quality standards, validation protocols, /complete integration

### Primary Agents (Use These First - Handle 80% Independently)
**Analytics Engineer** (`analytics-engineer-role`) - Owns transformation layer (dbt + Snowflake + BI data)
- SQL transformations, data modeling, performance optimization
- Business logic implementation, metrics, semantic layer
- Delegates to dbt-expert and snowflake-expert for complex cases

**Data Engineer** (`data-engineer-role`) - Owns ingestion layer (Orchestra + dlthub + Prefect + Airbyte)
- Pipeline setup and orchestration (batch AND streaming)
- Source system integration, data quality at ingestion
- Chooses right tool (dlthub vs Prefect) based on requirements
- Creates specialist agents when deep tool expertise needed

### Tool Specialists (Consultation Layer - 20% Edge Cases)
Role agents delegate to specialists who combine deep domain expertise with MCP tool access for informed, validated recommendations.

**Data Platform (Available)**:
- `dbt-expert`: SQL transformations, dbt patterns (MCP-enabled for dbt Cloud API)
- `snowflake-expert`: Warehouse optimization, cost analysis
- `tableau-expert`: BI optimization and dashboard performance
- `claude-code-expert`: Claude Code configuration specialist

**Note**: For other specialist needs (cloud infrastructure, business context, orchestration tools), create specialist agents using templates in `.claude/agents/specialists/specialist-template.md`.

**Pattern**: Role agents delegate when confidence <0.60 OR domain expertise needed
**Specialists**: Use MCP tools + expertise to provide validated, correct recommendations
**Correctness > Speed**: 15x token cost justified by significantly better outcomes (per Anthropic research)

### MCP Tool Execution Pattern

**Specialist agents provide MCP tool recommendations, main Claude executes them**:

1. **Specialist creates recommendation**:
   ```markdown
   ### RECOMMENDED MCP TOOL EXECUTION
   **Tool**: snowflake_query_manager
   **Query**: SELECT * FROM TASK_HISTORY WHERE STATE = 'FAILED'
   **Expected Result**: List of failed tasks
   **Fallback**: Direct Python execution if MCP unavailable
   ```

2. **Main Claude executes**:
   - Reads specialist recommendation
   - Executes MCP tool call or fallback approach
   - Returns results to specialist (if needed for analysis)
   - Implements specialist recommendations

3. **Why this pattern**:
   - Specialists focus on expertise (what to do, which tools to use)
   - Main Claude handles execution (MCP calls, Python scripts, Bash commands)
   - Specialists can't execute directly (research-only by design)
   - Clear separation of concerns: planning vs execution

*See `.claude/memory/patterns/cross-system-analysis-patterns.md` for detailed agent coordination*

## Skills System: Workflow Automation

### What are Skills?

**Skills** are reusable workflows that automate repetitive procedures through organized folders containing instructions, scripts, and resources. Skills complement the agent system by handling procedural automation, while agents provide domain expertise.

**Location**: `.claude/skills/[skill-name]/skill.md`
**Invocation**: Use the Skill tool to execute skill workflows

### When to Use Skills vs Agents vs Patterns

**Use Skills for**:
- ✅ Repeatable multi-step workflows (project setup, PR generation)
- ✅ Tool orchestration (coordinating Read, Write, Bash, Task tools)
- ✅ Template-driven document generation
- ✅ Process automation that you've done 3+ times manually

**Use Agents for**:
- ✅ Domain expertise and problem-solving (dbt optimization, AWS architecture)
- ✅ MCP-enhanced analysis (real-time data investigation)
- ✅ Complex decisions requiring research and recommendations
- ✅ Specialist consultation when confidence <0.60

**Use Patterns for**:
- ✅ Quick reference documentation (git workflows, delegation protocols)
- ✅ Decision frameworks ("when to use X vs Y")
- ✅ Production-validated solutions from completed projects
- ✅ Best practices and technical implementation patterns

**Complete Guide**: See `.claude/memory/patterns/knowledge-organization-strategy.md` for comprehensive decision framework

### Current Skill Status

**Implementation Status**: ✅ ACTIVE (4 high-value skills deployed)

**Available Skills**:
1. **project-setup** (`.claude/skills/project-setup/skill.md`)
   - Initialize new project structure with README, spec.md, context.md
   - Create git branch with proper naming conventions
   - Generate task tracking structure
   - **Saves**: 10-15 minutes per project setup

2. **pr-description-generator** (`.claude/skills/pr-description-generator/skill.md`)
   - Generate comprehensive PR descriptions from project context
   - Analyze git changes and create structured summary
   - Include test plan and quality checklist
   - **Saves**: 5-10 minutes per PR

3. **dbt-model-scaffolder** (`.claude/skills/dbt-model-scaffolder/skill.md`)
   - Generate dbt model boilerplate (staging, intermediate, mart)
   - Create schema.yml with tests and documentation
   - Follow dbt style guide and best practices
   - **Saves**: 15-20 minutes per model

4. **documentation-validator** (`.claude/skills/documentation-validator/skill.md`)
   - Validate documentation completeness before project closure
   - Check required files and sections exist
   - Verify internal links work
   - Generate comprehensive validation report
   - **Used by**: `/complete` command to ensure quality

**How to Use Skills**:
```
# Skills are invoked automatically when appropriate, or manually:
"Set up new project for [name]"  → project-setup skill
"Generate PR description"         → pr-description-generator skill
"Create dbt staging model"        → dbt-model-scaffolder skill
"Validate documentation"          → documentation-validator skill
```

**Integration**: Skills work alongside agents and reference patterns for comprehensive automation + expertise

### Skills + Agents + Patterns Architecture

```
Skills (.claude/skills/) - "HOW to execute workflows"
   ↓ Uses
Agents (.claude/agents/) - "WHO to consult for expertise"
   ↓ References
Patterns (.claude/memory/patterns/) - "WHAT solutions work"
```

**Example Flow**:
```
User: "Set up new dbt optimization project"
  ↓
Skill: project-setup
  - Creates project directories
  - Invokes analytics-engineer-role agent for technical approach
  - Generates README using agent recommendations + pattern templates
  - Creates git branch with appropriate naming
  ↓
Result: Structured project with expert guidance in 30 seconds
```

## Context Management & Memory System

### Session Start Protocol
1. **Recent Patterns** (`.claude/memory/recent/`) → Review last 30 days for similar solutions
2. **Domain Patterns** (`.claude/memory/patterns/`) → Load relevant architectural patterns
3. **Task Context** (`.claude/tasks/`) → Check for unfinished work
4. **Project Templates** (`.claude/memory/templates/`) → Use appropriate template

### Pattern Documentation Protocol
Use these markers in `.claude/tasks/*/findings.md` for automatic extraction:

```markdown
PATTERN: [Description of reusable pattern]
SOLUTION: [Specific solution that worked]
ERROR-FIX: [Error message] -> [Fix that resolved it]
ARCHITECTURE: [System design pattern]
INTEGRATION: [Cross-system coordination approach]
```

## Critical Rules

### Security & Git Workflow
**CRITICAL**: NEVER commit directly to protected branches (main, master, production, prod)
- Always create feature branch first
- All code changes require PR workflow
- Exception: da-agent-hub documentation-only changes

*See `.claude/memory/patterns/git-workflow-patterns.md` for complete git workflow patterns*

### Sandbox Principle
**CRITICAL**: `projects/active/<project-name>/` functions as an **isolated sandbox**

**All work stays in project folder until explicit deployment**:
- Analysis, code, documentation, testing → `projects/active/<project>/`
- Never write to production repos during active development
- Read-only access to production for comparison only

**Deploy only when user explicitly requests**:
- "Deploy this to [repo]"
- "Push to production"
- "Create PR in [repo]"
- Project finalized with `/finish` command

### Development Best Practices
- **Always start from up-to-date main branch**: Run `git checkout main && git pull origin main` before starting any work
- **DO NOT MOVE FORWARD until you've fixed a problem**: If blocked on step 1, stop and fix completely before step 2
- **Git branches**: Prefix with `feature/` or `fix/`
- **Use subagents**: Optimize context window with task delegation
- **Preserve context links**: Maintain connections between ideas → projects → operations

## Task vs Project Classification

### Use Project Structure When:
- Multi-day efforts spanning multiple work sessions
- Cross-repository coordination (dbt + snowflake + tableau)
- Research and analysis informing multiple decisions
- Collaborative work with team members/reviewers
- Knowledge preservation needed for future reference
- Complex troubleshooting requiring systematic investigation

### Use Simple Task Execution When:
- Quick fixes (typos, small config changes, single-file updates)
- Immediate responses to questions or information requests
- One-off scripts or utilities
- Documentation updates without research
- Status checks or system diagnostics
- Simple file operations or code formatting

## Context Clarity & File Reference System

### Visual File Reference Indicators
- **📁 PROJECT**: Working files in `projects/active/<project-name>/`
- **📦 REPO**: Source repository files (original/production versions)
- **🎯 DEPLOY**: Deployment target locations

### Context Declaration Protocol
Before analysis, declare context assumptions:

```
📍 Context Check:
- Working File: 📁 PROJECT projects/active/feature-x/app.py
- Reference: 📦 REPO ../original-repo/app.py
- Deploy Target: 🎯 DEPLOY production-repo/

If you want different sources, please redirect me.
```

## Smart Repository Context Resolution

### Automatic Owner/Repo Detection
When working with GitHub repositories, use smart context resolution to avoid specifying owner repeatedly:

```bash
# Resolve repository context from config/repositories.json
python3 scripts/resolve-repo-context.py dbt_cloud
# Output: YOUR_ORG dbt_cloud

# Use resolved context in GitHub MCP operations
mcp__github__list_issues owner="YOUR_ORG" repo="dbt_cloud"
```

### Available Commands
- `python3 scripts/resolve-repo-context.py <repo_name>` - Get owner and repo
- `python3 scripts/resolve-repo-context.py --json <repo_name>` - Get full context as JSON
- `python3 scripts/resolve-repo-context.py --list` - List all resolvable repositories
- `./scripts/get-repo-owner.sh <repo_name>` - Get just the owner (bash helper)

### Agent Integration
All specialist agents working with GitHub should:
1. Resolve repository context before GitHub MCP calls
2. Use explicit owner/repo parameters in all GitHub MCP operations
3. Reference pattern documentation: `.claude/memory/patterns/github-repo-context-resolution.md`

**Benefit**: Eliminates cognitive overhead of remembering organization name for every GitHub operation while maintaining explicit, correct MCP calls.

## Knowledge Repository Structure

### Platform Documentation
`knowledge/platform/` - ADLC Framework platform documentation organized by lifecycle phases:

- **`planning/`** - ADLC Plan Phase (idea management, strategic planning, GitHub issue workflows)
- **`development/`** - ADLC Develop/Test/Deploy (agent development, VS Code integration, context management)
- **`operations/`** - ADLC Operate/Observe/Discover/Analyze (cross-repo coordination, troubleshooting)
- **`architecture/`** - System design, agent capabilities, confidence routing patterns
- **`mcp-servers/`** - MCP integration guides (dbt, AWS docs, Slack, filesystem)
- **`specialists/`** - Specialist agent documentation and patterns
- **`training/`** - Agent learning, chat analysis, continuous improvement

**See `knowledge/platform/README.md` for complete navigation guide**

### Extending the Knowledge Base

As you build your AI-augmented workflow, create additional knowledge directories for your needs:

**Example: Team Documentation**
```
knowledge/team/
├── architecture/          # Your data architecture docs
├── integrations/          # System integration guides
└── products/              # Product/service documentation
```

**Example: Application Documentation**
```
knowledge/applications/
└── <app-name>/
    ├── architecture/      # System design, data flows
    ├── deployment/        # Deployment runbooks
    └── operations/        # Monitoring, troubleshooting
```

**Three-Tier Documentation Pattern** (Recommended for Production Applications):

**Tier 1: Repository README** (Lightweight, Developer-Focused)
- **Location**: `<your-app-repo>/README.md`
- **Purpose**: Get developers productive fast
- **Contains**: App purpose, local dev setup, commands, link to knowledge base
- **Size**: < 200 lines

**Tier 2: Knowledge Base** (Comprehensive, Cross-System)
- **Location**: `knowledge/applications/<app-name>/`
- **Purpose**: Complete reference for deployment, operations, architecture
- **Audience**: AI agents coordinating deployments/operations

**Tier 3: Agent Pattern Index** (Pointers + Confidence)
- **Location**: `.claude/agents/specialists/<agent>.md`
- **Purpose**: Help agents find proven patterns quickly
- **Contains**: Pattern name, confidence score, link to Tier 2 docs

**Key Principles**:
- **Single Source of Truth**: Each piece of info lives in ONE canonical location
- **Cross-Reference, Don't Duplicate**: Use links instead of copying content
- **Task-Aware Discovery**: Agents read what they need, when they need it
- **Layer-Appropriate Detail**: Match detail level to audience needs

## Git Workflow & Security Rules

### Critical Security Rules - Protected Branches

**NEVER commit directly to**: `main`, `master`, `production`, `prod`, `release/*`, `hotfix/*`

**Mandatory Workflow**:
1. **ALWAYS create feature branch** before making any code changes
2. **ALWAYS create Pull Request** for code review and approval
3. **NEVER push directly** to protected branches
4. **NEVER merge without approval** (except for documentation-only changes)

**Pre-Commit Safety Check** - Claude MUST verify branch before committing:
```bash
CURRENT_BRANCH=$(git branch --show-current)
PROTECTED_BRANCHES=("main" "master" "production" "prod")

for branch in "${PROTECTED_BRANCHES[@]}"; do
    if [ "$CURRENT_BRANCH" = "$branch" ]; then
        echo "❌ ERROR: Cannot commit to protected branch '$CURRENT_BRANCH'"
        echo "Please create a feature branch: git checkout -b feature/your-feature-name"
        exit 1
    fi
done
```

**If user requests commit to protected branch**:
1. **Stop immediately** - Do not execute the commit
2. **Explain the security policy** - Protected branches require PR workflow
3. **Offer to create feature branch** - Suggest branch name based on work
4. **Create PR after commit** - Ensure changes go through approval process

### Branch Naming Conventions

**Standard prefixes**:
- `feature/[description]` - New features
- `fix/[description]` - Bug fixes
- `docs/[description]` - Documentation updates
- `refactor/[description]` - Code refactoring
- `test/[description]` - Testing improvements

**Best practices**: Use descriptive, kebab-case names (e.g., `feature/add-customer-dashboard`)

### Repository-Specific Branch Structures

**dbt_cloud**: master (prod), dbt_dw (staging) - Branch from dbt_dw
**dbt_errors_to_issues**: main (prod) - Branch directly from main
**roy_kent**: master (prod) - Branch directly from master
**sherlock**: main (prod) - Branch directly from main

### Standard Workflow

**CRITICAL - Always start from up-to-date main**:
```bash
git checkout main
git pull origin main
git checkout -b feature/your-feature-name
```

**Complete workflow checklist**:
1. ✅ Sync with main branch before creating features
2. ✅ Create descriptive branch names
3. ✅ Keep branches focused and atomic
4. ✅ Test locally before pushing
5. ✅ Create PR with clear description
6. ✅ Wait for approval
7. ✅ Clean up branches after merge

## ADLC Continuous Improvement Strategy

### During Project Completion
Actively identify and capture:
- **Agent Capability Enhancements**: New tool patterns, integration strategies, troubleshooting insights
- **Agent File Updates**: Novel patterns for specialist agents (`.claude/agents/`)
- **Knowledge Base Enhancement**: System architecture patterns, process improvements (`knowledge/`)

### Knowledge Extraction Locations

When completing projects, extract learnings to appropriate locations based on content type:

**Production Application Knowledge** → `knowledge/applications/<app-name>/`
- **When**: Deploying new apps or major app updates
- **Structure**: Three-tier pattern (Tier 2 - comprehensive docs)
  - `architecture/` - System design, data flows, infrastructure details
  - `deployment/` - Complete deployment runbooks, Docker builds, AWS configuration
  - `operations/` - Monitoring, troubleshooting guides, incident response
- **Examples**: ALB OIDC authentication, ECS deployment patterns, multi-service Docker
- **Updates Required**:
  1. Create/update knowledge base docs for the application
  2. Update agent pattern index (e.g., aws-expert.md with confidence scores)
  3. Add to "Known Applications" in relevant role agents (e.g., ui-ux-developer-role.md)
  4. Create lightweight README in actual repo (Tier 1) linking to knowledge base

**Platform/Tool Patterns** → `knowledge/platform/`
- **When**: Discovering reusable patterns for ADLC workflow
- **Structure**: Organized by ADLC phase (planning/, development/, operations/)
- **Examples**: Testing frameworks, git workflows, cross-system analysis patterns

**Agent Expertise** → `.claude/agents/specialists/<agent>.md`
- **When**: Validating patterns in production, improving confidence scores
- **Updates**: Add production-validated patterns section, update confidence levels
- **Reference**: Link to knowledge base (Tier 2) for full implementation

**Role Agent Applications** → `.claude/agents/roles/<role>.md`
- **When**: New applications deployed that role will work on
- **Updates**: Add to "Known Applications" section with stack, patterns, key learnings
- **Purpose**: Help agents quickly find context when tasked with app work

### Improvement PR Decision Framework
Create separate improvement PRs for:
- **HIGH IMPACT**: Agent updates benefiting multiple future projects
- **KNOWLEDGE GAPS**: Missing documentation causing repeated research
- **PROCESS OPTIMIZATION**: Workflow improvements with measurable efficiency gains
- **INTEGRATION ENHANCEMENT**: Cross-tool coordination improvements
- **ADLC METHODOLOGY**: Core system workflow refinements

**Examples**:
- "feat: Enhance aws-expert with ALB OIDC production patterns from sales-journal deployment"
- "docs: Add React + FastAPI application architecture to knowledge/applications/"
- "feat: Document three-tier documentation pattern for future app deployments"

## Agent Training & Learning System

### Chat Analysis Features
- User-agnostic discovery of Claude conversations
- Privacy-preserving local analysis
- Effectiveness metrics and improvement recommendations
- Integration with `/complete` command for automatic learning extraction

**Usage**: `./scripts/analyze-claude-chats.sh`

**Results**: `knowledge/platform/training/analysis-results/` (local only)

### Continuous Learning Loop
```
🔧 PROJECT WORK → 💬 CONVERSATIONS → 📊 ANALYSIS
    ↑                                      ↓
🚀 ENHANCED AGENTS ← 📝 IMPROVEMENT PRs ← 💡 RECOMMENDATIONS
```

## Complete Development Workflow

```
💡 CAPTURE: /capture → GitHub issue creation
    ↓ (Use GitHub for prioritization)
🔬 RESEARCH: /research [text|issue#] → Deep exploration → Feasibility → Technical approach
    ↓ Informed decision-making
🚀 START: /start [issue#|"text"] → project setup → worktree creation → specialist agents → development
    ↓ Deploy to production
✅ COMPLETE: /complete → archive → worktree cleanup → close GitHub issue → next iteration
```

## Additional Resources

**Git Workflows**: `.claude/memory/patterns/git-workflow-patterns.md`
**Testing Patterns**: `.claude/memory/patterns/testing-patterns.md`
**Cross-System Analysis**: `.claude/memory/patterns/cross-system-analysis-patterns.md`
**VS Code Worktrees**: `knowledge/platform/development/vscode-worktree-integration.md`
**Agent Definitions**: `.claude/agents/`
**Platform Documentation**: `knowledge/platform/README.md`
