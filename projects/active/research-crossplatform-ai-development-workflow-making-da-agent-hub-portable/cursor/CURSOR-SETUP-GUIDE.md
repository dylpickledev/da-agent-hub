# DA Agent Hub - Cursor Setup Guide

**Version**: 1.0
**Target Tool**: Cursor (0.40+)
**Platform**: macOS, Linux, Windows (WSL)

---

## Overview

This guide walks you through setting up the DA Agent Hub ADLC workflow in Cursor, enabling the same powerful analytics development capabilities as Claude Code through Cursor's native features.

**What You'll Get**:
- ✅ Full ADLC workflow (/research, /capture, /start, /switch, /complete)
- ✅ Role-based agent coordination with specialists
- ✅ File-based specialist delegation via Composer
- ✅ GitHub Issues integration for idea management
- ✅ Automated knowledge extraction and learning
- ✅ Project structure and git workflow automation

---

## Prerequisites

### Required Tools

1. **Cursor** (0.40 or later)
   - Download: https://cursor.sh/
   - Verify: `cursor --version`

2. **GitHub CLI** (`gh`)
   - macOS: `brew install gh`
   - Linux: https://github.com/cli/cli#installation
   - Windows: `winget install GitHub.cli`
   - Verify: `gh --version`
   - Authenticate: `gh auth login`

3. **Git**
   - Verify: `git --version`
   - Configure: `git config --global user.name "Your Name"`
   - Configure: `git config --global user.email "your@email.com"`

4. **Bash** (Unix shell)
   - macOS/Linux: Built-in
   - Windows: Use WSL (Windows Subsystem for Linux)

### Optional (Recommended)

5. **jq** (JSON parsing for scripts)
   - macOS: `brew install jq`
   - Linux: `sudo apt install jq` or `sudo yum install jq`
   - Windows: Available in WSL

---

## Installation Steps

### Step 1: Clone DA Agent Hub Repository

If you don't already have it:

```bash
cd ~/github_personal/  # Or your preferred location
git clone https://github.com/dylpickledev/da-agent-hub.git
cd da-agent-hub
```

If you have it, ensure it's up to date:

```bash
cd da-agent-hub
git checkout main
git pull origin main
```

### Step 2: Install Cursor .cursorrules

Copy the Cursor adapter configuration to your DA Agent Hub repository root:

```bash
# From this research project
cp projects/active/research-crossplatform-ai-development-workflow-making-da-agent-hub-portable/cursor/.cursorrules ./.cursorrules

# Verify it's there
ls -la .cursorrules
```

**What this does**:
- Defines all DA Agent Hub commands (/research, /capture, /start, etc.)
- Configures agent coordination protocols
- Sets up Composer context management
- Provides workflow guidance and best practices

### Step 3: Make Scripts Executable

Ensure all workflow scripts have execute permissions:

```bash
chmod +x ./scripts/*.sh

# Verify
ls -l ./scripts/
```

You should see scripts like:
- `capture.sh` - Idea capture
- `research.sh` - Research projects
- `start.sh` - Project initialization
- `switch.sh` - Context switching
- `finish.sh` - Project completion

### Step 4: Configure GitHub Integration

The DA Agent Hub uses GitHub Issues for idea management:

```bash
# Authenticate GitHub CLI (if not already done)
gh auth login

# Verify GitHub repository connection
gh repo view

# Create required labels (if they don't exist)
gh label create "idea" --description "Ideas for future development" --color "d4c5f9" || true
gh label create "in-progress" --description "Work in progress" --color "fbca04" || true
gh label create "bi-analytics" --description "BI and analytics work" --color "0075ca" || true
gh label create "data-engineering" --description "Data pipeline work" --color "1d76db" || true
gh label create "analytics-engineering" --description "dbt and transformation work" --color "0e8a16" || true
gh label create "architecture" --description "Platform and architecture" --color "5319e7" || true
gh label create "ui-development" --description "UI and frontend work" --color "f9d0c4" || true
```

### Step 5: Set Up Git Worktrees (Optional but Recommended)

Worktrees enable clean project isolation:

```bash
# Run setup script
./scripts/setup-worktrees.sh

# This creates:
# - ../da-agent-hub-worktrees/ directory
# - Configures VS Code workspace settings (also works with Cursor)
```

**Benefits**:
- Each project gets its own working directory
- No interference between projects
- Clean context switching with /switch command

### Step 6: Open Repository in Cursor

```bash
# Open DA Agent Hub in Cursor
cursor .

# Or from Cursor: File → Open Folder → Select da-agent-hub/
```

### Step 7: Verify Setup

In Cursor, open a new Composer chat and test:

```
Type: /capture "test idea for cursor setup"
```

**Expected Result**:
- Cursor executes `./scripts/capture.sh "test idea for cursor setup"`
- GitHub issue created
- You see: ✅ GitHub issue created successfully!

**Clean up test**:
```bash
# Close the test issue
gh issue list --label idea
gh issue close <issue-number>
```

---

## Configuration

### Cursor Settings (Optional Enhancements)

Open Cursor Settings (Cmd/Ctrl + ,):

**Composer Context Management**:
```json
{
  "cursor.composer.maxContextFiles": 10,
  "cursor.composer.includeFolders": [
    "projects/active",
    ".claude/agents",
    ".claude/memory"
  ]
}
```

**File Watching** (for context.md updates):
```json
{
  "files.watcherExclude": {
    "**/projects/completed/**": true,
    "**/.git/objects/**": true,
    "**/.git/subtree-cache/**": true
  }
}
```

### Repository-Specific Settings

Create `.vscode/settings.json` (Cursor uses these too):

```json
{
  "files.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/projects/completed": false
  },
  "search.exclude": {
    "**/projects/completed": true,
    "**/.git": true
  },
  "files.associations": {
    ".cursorrules": "markdown",
    "*.md": "markdown"
  }
}
```

---

## Usage Guide

### Basic Workflow

#### 1. Capture an Idea

```
In Cursor Composer:
/capture "Build customer analytics dashboard with real-time metrics"
```

**What Happens**:
- GitHub issue created with 'idea' label
- Auto-categorized (bi-analytics, data-engineering, etc.)
- Stored for prioritization

#### 2. Start a Project

**From existing issue**:
```
/start 85
```

**From scratch** (creates issue automatically):
```
/start "Fix broken dashboard filters"
```

**What Happens**:
- Project structure created in `projects/active/feature-*/`
- Files loaded into Composer: README.md, spec.md, context.md
- GitHub issue linked and labeled 'in-progress'
- Git branch created

#### 3. Work with Agents

**Use role agents for primary work**:
```
@.claude/agents/roles/analytics-engineer-role.md

"I need to optimize this incremental dbt model for better performance"
```

**Delegate to specialists for complex tasks**:

**Step 1 - Main agent writes request**:
```
[Agent creates: tasks/dbt-expert-request.md with problem context]

"Please consult dbt-expert for this optimization"
```

**Step 2 - Invoke specialist in Composer**:
```
@.claude/agents/specialists/dbt-expert.md

"Please analyze tasks/dbt-expert-request.md and provide optimization recommendations"
```

**Step 3 - Specialist analyzes and writes findings**:
```
[dbt-expert creates: tasks/dbt-expert-findings.md with recommendations]
```

**Step 4 - Main agent reads and executes**:
```
@.claude/agents/roles/analytics-engineer-role.md
@tasks/dbt-expert-findings.md

"Implement the dbt-expert recommendations"
```

#### 4. Switch Between Projects

**Save current work and switch**:
```
/switch
```

**Switch to specific branch**:
```
/switch feature-other-project
```

**What Happens**:
- Current work committed (WIP)
- Pushed to remote (backup)
- Clean switch to new branch

#### 5. Complete Project

```
/complete feature-customer-analytics-dashboard
```

**What Happens** (Two-Phase):

**Phase 1 - Analysis**:
- Reads all project files
- Proposes knowledge updates
- Shows exact changes to agent files
- **Requests approval**

**Phase 2 - Execution** (after you approve):
- Updates agent definitions
- Archives project to `projects/completed/`
- Closes GitHub issue
- Extracts patterns to memory

### Advanced Workflows

#### Research Before Building

```
# Deep analysis of fuzzy idea
/research "Should we migrate from Prefect to Orchestra for orchestration?"

# After analysis in projects/active/research-*/
# Decide: capture for roadmap OR start immediately

/capture "findings from research"
# OR
/start "Implement Orchestra migration based on research"
```

#### Multi-Day Project with Pauses

```
# Day 1
/start 87
[Work on implementation]
/pause "Implemented data models, next: add tests and documentation"

# Day 2 (new Cursor session)
# Open project directory
# Load context: @spec.md @context.md @tasks/current-task.md
# Continue where you left off
```

#### Context Switching for Urgent Work

```
# Working on feature A
/start 78
[Implementing dashboard]

# Urgent bug!
/switch
/start "Fix critical data pipeline failure"
[Fix bug, deploy]
/complete fix-critical-data-pipeline-failure

# Return to original work
/switch feature-new-bi-metrics-dashboard
[Continue dashboard work]
```

---

## Agent Coordination in Cursor

### Role Agents (Primary - Use First)

Located in `.claude/agents/roles/`:

**analytics-engineer-role** - SQL, dbt, data modeling
- Use for: Transformations, metrics, dimensional modeling
- Confidence: 0.85+ handles directly, <0.60 delegates to specialists

**data-engineer-role** - Pipelines, orchestration, ingestion
- Use for: Pipeline setup, source integration, workflow coordination
- Confidence: Similar delegation pattern

**data-architect-role** - System design, strategic decisions
- Use for: Architecture choices, platform roadmap, cross-system design

### Specialist Agents (Consultation - Use for Edge Cases)

Located in `.claude/agents/specialists/`:

**dbt-expert** - Advanced dbt patterns
- When: Complex macros, advanced incremental strategies, dbt architecture

**snowflake-expert** - Warehouse optimization
- When: Cost optimization, query performance tuning, Snowflake-specific features

**tableau-expert** - BI optimization
- When: Dashboard performance, visualization best practices

**dlthub-expert** - Data ingestion
- When: Complex ingestion patterns, source system integration

### File-Based Coordination Pattern

**When to use**:
- Main agent confidence <0.60
- Complex problem requiring deep expertise
- Need detailed analysis and recommendations

**Protocol**:
1. **Main agent writes**: `tasks/[specialist]-request.md`
2. **You invoke specialist**: `@.claude/agents/specialists/[specialist].md`
3. **Specialist writes**: `tasks/[specialist]-findings.md`
4. **Main agent reads**: `@tasks/[specialist]-findings.md`
5. **Main agent executes**: Implements recommendations

**When NOT to use**:
- Simple questions (just @ mention specialist inline)
- Main agent confidence ≥0.60 (handle directly)
- Quick consultations (no delegation protocol needed)

---

## Cursor-Specific Features

### Composer Multi-File Editing

**Leverage Composer for**:
- Editing spec.md and implementation files simultaneously
- Viewing context.md while coding
- Applying specialist recommendations across multiple files
- Coordinated changes to related components

**Example**:
```
@spec.md @src/models/customer_churn.sql @tests/test_customer_churn.py

"Implement the customer churn model based on spec.md requirements,
 including tests"
```

### @ Mentions for Context

**Load project files**:
```
@spec.md @context.md @tasks/current-task.md
```

**Load agent context**:
```
@.claude/agents/roles/analytics-engineer-role.md
```

**Load memory patterns**:
```
@.claude/memory/patterns/dbt-incremental-optimization-pattern.md
```

**Load specialist**:
```
@.claude/agents/specialists/dbt-expert.md
```

### Chat History & Context

**Preserve context across sessions**:
- Cursor maintains chat history
- Reference previous conversations
- Load same files to resume work

**Best Practice**:
- Update `context.md` before ending session
- Document decisions and next steps
- Load context.md when resuming

---

## Troubleshooting

### Commands Not Recognized

**Symptom**: Typing `/capture` does nothing

**Solutions**:
1. Verify `.cursorrules` exists in repository root: `ls .cursorrules`
2. Reload Cursor window: Cmd/Ctrl + Shift + P → "Reload Window"
3. Check Cursor settings: Rules file should be detected
4. Try closing and reopening the project

### Scripts Not Executing

**Symptom**: "Permission denied" when running commands

**Solutions**:
```bash
# Make scripts executable
chmod +x ./scripts/*.sh

# Verify permissions
ls -l ./scripts/
```

### GitHub Integration Issues

**Symptom**: Issues not creating or `gh` command not found

**Solutions**:
```bash
# Install GitHub CLI
brew install gh  # macOS
# Or follow: https://github.com/cli/cli#installation

# Authenticate
gh auth login

# Verify repository connection
cd da-agent-hub
gh repo view
```

### Agent Files Not Loading

**Symptom**: `@.claude/agents/...` doesn't work

**Solutions**:
1. Use full path from repository root:
   ```
   @.claude/agents/specialists/dbt-expert.md
   ```
2. Verify file exists:
   ```bash
   ls .claude/agents/specialists/dbt-expert.md
   ```
3. Check file is readable (not empty)

### Worktree Issues

**Symptom**: `/switch` fails or worktrees not created

**Solutions**:
```bash
# Re-run worktree setup
./scripts/setup-worktrees.sh

# Verify worktree directory
ls ../da-agent-hub-worktrees/

# Check git worktree list
git worktree list
```

### Project Files Not Loading

**Symptom**: Composer doesn't show project files after `/start`

**Solutions**:
1. Manually load in Composer:
   ```
   @spec.md @context.md @README.md
   ```
2. Navigate to project directory in Cursor sidebar
3. Verify files exist:
   ```bash
   ls projects/active/feature-*/
   ```

---

## Tips & Best Practices

### Workflow Efficiency

1. **Use /start for all projects**: Gets proper context automatically
2. **Update context.md frequently**: Helps with /pause and /switch
3. **Leverage Composer**: Edit multiple files simultaneously
4. **Keep project files in context**: spec.md, context.md, tasks/

### Agent Coordination

1. **Start with role agents**: They handle 80% of work
2. **Delegate consciously**: Only use specialists for edge cases (<0.60 confidence)
3. **Write clear requests**: Provide full context in delegation files
4. **Validate specialist output**: Understand recommendations before implementing

### Git & GitHub

1. **Always pull main before /start**: Avoid conflicts
   ```bash
   git checkout main && git pull origin main
   ```
2. **Commit frequently**: Small, atomic commits during work
3. **Use /switch for context changes**: Preserves work, no loss
4. **Create PRs for review**: After /complete, use `gh pr create`

### Knowledge Preservation

1. **Document patterns in task findings**: Use PATTERN:, SOLUTION:, ERROR-FIX: markers
2. **Let /complete extract learnings**: Automatic pattern extraction
3. **Reference memory patterns**: Use @.claude/memory/patterns/ for proven solutions
4. **Update agents after projects**: Approve knowledge updates from /complete

---

## Next Steps

### Getting Started

1. ✅ **Complete setup** (this guide)
2. **Try a test project**:
   ```
   /capture "Test project for Cursor setup validation"
   /start 1
   [Make a simple change]
   /complete feature-test-project-for-cursor-setup-validation
   ```
3. **Explore agent coordination**:
   ```
   @.claude/agents/roles/analytics-engineer-role.md
   "Help me understand how to use specialist delegation"
   ```
4. **Review knowledge base**:
   ```
   @knowledge/da-agent-hub/README.md
   ```

### Learning Resources

**Platform Documentation**:
- `knowledge/da-agent-hub/README.md` - Complete platform guide
- `knowledge/da-agent-hub/development/` - Development workflows
- `.claude/memory/patterns/` - Proven patterns and solutions

**Agent Definitions**:
- `.claude/agents/roles/` - Role agent capabilities
- `.claude/agents/specialists/` - Specialist expertise areas

**Example Projects**:
- `projects/completed/` - See archived projects for examples

### Getting Help

**Common Questions**:
1. Load platform docs: `@knowledge/da-agent-hub/README.md`
2. Check troubleshooting section above
3. Review `.cursorrules` for command documentation
4. Examine bash scripts: `./scripts/` for implementation details

**Community**:
- GitHub Issues: https://github.com/dylpickledev/da-agent-hub/issues
- Discussions: https://github.com/dylpickledev/da-agent-hub/discussions

---

## Comparison: Cursor vs Claude Code

### What's the Same
- ✅ Full ADLC workflow (all commands)
- ✅ Agent coordination and delegation
- ✅ Project structure and organization
- ✅ Knowledge extraction and learning
- ✅ GitHub integration
- ✅ Git workflow automation

### What's Different

**Cursor Advantages**:
- ✅ Better multi-file editing (Composer)
- ✅ Familiar VS Code interface
- ✅ Native git integration
- ✅ Extensive ecosystem of extensions

**Claude Code Advantages**:
- ✅ Automatic subagent spawning (Task tool)
- ✅ Built-in TodoWrite for task tracking
- ✅ Native /pause with conversation state
- ✅ Integrated MCP tool support

**Cursor Adaptations**:
- **Agent coordination**: File-based instead of Task tool (works well!)
- **Task tracking**: Use context.md + Cursor tasks (simple, effective)
- **Pause/resume**: context.md updates + chat history (manual but reliable)
- **MCP tools**: Not yet available (specialists still provide expertise)

### Migration Notes

**From Claude Code to Cursor**:
- Most workflows identical (same bash scripts)
- Agent coordination slightly more manual (file-based)
- All knowledge and memory system unchanged
- Projects structure identical

**Using Both Tools**:
- Projects are tool-agnostic (markdown files)
- Can switch between tools seamlessly
- Knowledge base shared
- Git workflow compatible

---

## Changelog

### Version 1.0 (2025-10-16)
- Initial Cursor adapter release
- Complete .cursorrules implementation
- File-based agent coordination protocol
- Setup guide and documentation
- Validated with test workflows

---

*You now have the complete DA Agent Hub ADLC workflow in Cursor! Start with `/capture` to brainstorm ideas, or `/start` to begin building immediately.*
