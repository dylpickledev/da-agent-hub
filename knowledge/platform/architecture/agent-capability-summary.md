# Agent Capability Summary: Claude ADLC Framework

## Overview
The Claude ADLC Framework provides a foundational AI-powered Analytics Development Lifecycle (ADLC) platform with **3 role-based agents**, **4 specialist agents**, and **1 MCP server integration** - designed as a starting point that you can extend for your specific data stack.

---

## 🎭 Role-Based Agents (Primary Layer)

### Included Roles
1. **Analytics Engineer** (`analytics-engineer-role`) - SQL transformations, dbt modeling, data warehouse optimization
2. **Data Engineer** (`data-engineer-role`) - Pipeline orchestration, source integration, data ingestion
3. **Onboarding Agent** (`onboarding-agent`) - Setup assistance, MCP configuration, ADLC workflow guidance

**Delegation Pattern**: Roles handle 80% of work independently, delegate to specialists for deep technical expertise (20% of cases)

---

## 🔧 Tool Specialists (Consultation Layer)

### Included Specialists
- **dbt-expert**: SQL transformations, dbt Cloud patterns via `dbt-mcp`
- **snowflake-expert**: Warehouse optimization, cost analysis (uses dbt-mcp for queries)
- **tableau-expert**: Dashboard optimization, BI best practices
- **claude-code-expert**: Claude Code configuration, agent design, MCP integration patterns

**Research-Only Mode**: All specialists provide expert recommendations but don't execute directly. Main Claude implements their guidance.

---

## 🔌 MCP Server Integrations

### Included MCP Servers
- **dbt-mcp** (`@dylpickle/dbt-cloud-mcp`): dbt Cloud API integration
  - Run models, inspect schemas, execute queries
  - Job management and metadata discovery
  - See: `knowledge/platform/mcp-servers/dbt-mcp-integration-guide.md`

### Adding More MCP Servers
The framework is designed to easily integrate additional MCP servers for your stack:

**Cloud & Infrastructure Examples**:
- `aws-mcp`: AWS service management
- `azure-mcp`: Azure platform integration

**Data Platform Examples**:
- `snowflake-mcp`: Direct warehouse queries (vs dbt-mcp proxy)
- `postgres-mcp`: PostgreSQL database access
- `bigquery-mcp`: Google BigQuery integration

**Development & Collaboration Examples**:
- `github-mcp`: Repository operations, PR workflows
- `slack-mcp`: Team notifications and alerts
- `filesystem-mcp`: Advanced file operations

See `.claude/mcp.json` for configuration and `knowledge/platform/mcp-servers/` for integration guides.

---

## 🚀 Key Capabilities

### 1. Intelligent Delegation
Agents use confidence-based routing to decide when to consult specialists:
```
Role Agent (e.g., Analytics Engineer)
    ↓ Confidence ≥0.85: Handle directly (80% of work)
    ↓ Confidence <0.60: Delegate to specialist (20% of work)
Specialist (e.g., dbt-expert)
    ↓ Provides expert recommendation using MCP tools
Main Claude executes recommendation
```

### 2. Extensible Architecture
The framework provides templates for adding:
- **New Role Agents**: `.claude/agents/roles/role-template.md`
- **New Specialists**: `.claude/agents/specialists/specialist-template.md`
- **New MCP Servers**: `knowledge/platform/mcp-servers/` integration guides

### 3. Production-Validated Patterns
Specialists maintain confidence scores for proven patterns:
- **dbt Incremental Models** (confidence: 0.90) - Production-tested performance
- **Snowflake Query Optimization** (confidence: 0.85) - Cost reduction patterns
- **Dashboard Troubleshooting** (confidence: 0.88) - Systematic debugging approach

---

## 📈 Extending Your Platform

### Adding New Roles
Create role agents for your team's needs:
- **BI Developer**: Dashboard design, enterprise reporting
- **Data Architect**: Platform strategy, system design
- **QA Engineer**: Testing strategies, quality validation
- **MLOps Engineer**: Model deployment, monitoring

Use `.claude/agents/roles/role-template.md` as starting point.

### Adding New Specialists
Create specialists for your specific tools:
- **airflow-expert**: Orchestration patterns
- **fivetran-expert**: Data replication best practices
- **looker-expert**: LookML modeling and optimization
- **your-tool-expert**: Custom internal tooling

Use `.claude/agents/specialists/specialist-template.md` as starting point.

### Adding New MCP Servers
Integrate your critical tools via MCP:
1. Find or create MCP server for your tool
2. Add configuration to `.claude/mcp.json`
3. Create integration guide in `knowledge/platform/mcp-servers/`
4. Update specialist agents to use new MCP capabilities

---

## 🎯 Design Philosophy

**Start Small, Grow Deliberately**:
- Included agents cover core data workflows (dbt + BI)
- Templates make it easy to add your specific tools
- Patterns show how mature systems scale
- Focus on what you actually use, not everything possible

**Quality Over Quantity**:
- Each agent should solve real problems for your workflow
- Specialists should have deep expertise in their domain
- MCP integrations should enable capabilities you can't get otherwise

**Learn by Building**:
- The framework is designed to teach AI-augmented workflows
- Templates and examples show patterns to follow
- Documentation explains not just "what" but "why"

---

## 📚 Documentation Structure

- **Agent Definitions**: `.claude/agents/` - All role and specialist agent configurations
- **MCP Integration**: `knowledge/platform/mcp-servers/` - Server setup and usage guides
- **Architecture Patterns**: `knowledge/platform/architecture/` - System design and delegation patterns
- **Development Guides**: `knowledge/platform/development/` - Building custom agents and skills

---

**Version**: 2.0.0 (Public Framework Edition)
**Last Updated**: 2025-10-25
**Focus**: Starter platform that teaches extensibility patterns
