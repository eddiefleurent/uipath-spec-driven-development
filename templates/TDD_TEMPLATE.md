# Technical Design Document: [Project Name]

**Version:** 1.0  
**Status:** Draft | In Development | Production

---

## 1. Project Overview

### 1.1 Business Process

| Property | Value |
|----------|-------|
| **Process Name** | [Name] |
| **Business Owner** | [Name/Team] |
| **Description** | [Brief description of what the automation does] |
| **Trigger** | [Scheduled / Queue / Manual / Event] |
| **Frequency** | [How often it runs] |

### 1.2 Scope

**In Scope:**
- [Item 1]
- [Item 2]

**Out of Scope:**
- [Item 1]
- [Item 2]

---

## 2. Architecture

### 2.1 Pattern

| Property | Value |
|----------|-------|
| **Pattern** | [REFramework / Dispatcher-Performer / Linear / Custom] |
| **Rationale** | [Why this pattern was chosen] |

### 2.2 Component Diagram

```
[Main Process]
├── [Workflow 1]
├── [Workflow 2]
│   ├── [Sub-workflow 2.1]
│   └── [Sub-workflow 2.2]
└── [Workflow 3]
```

### 2.3 Data Flow

```
[Input Source] → [Processing] → [Output Destination]
```

---

## 3. Workflows

### 3.1 Workflow Inventory

| Workflow | Purpose | Type | Invoked By |
|----------|---------|------|------------|
| Main.xaml | Entry point | Orchestrator | Orchestrator Trigger |
| [Workflow].xaml | [Purpose] | [Atomic/Framework] | [Parent] |

### 3.2 Workflow Details

#### [WorkflowName].xaml

| Property | Value |
|----------|-------|
| **Purpose** | [One sentence description] |
| **Type** | Atomic / Framework / Orchestrator |
| **Invoked By** | [Parent workflow or trigger] |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_Argument | In | String | - | [Description] |
| out_Result | Out | Boolean | - | [Description] |

**Variables:**

| Name | Type | Scope | Description |
|------|------|-------|-------------|
| str_Variable | String | Main Sequence | [Purpose] |

**Activities Used:**
- [Activity 1]
- [Activity 2]

**Logic Summary:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| [ErrorType] | [How it's handled] |

---

## 4. Tests

### 4.1 Test Inventory

| Test | Type | Tests | Status |
|------|------|-------|--------|
| Test_[Name] | Unit | [Workflow] | Pass/Fail/Pending |

### 4.2 Test Coverage

| Workflow | Unit Tests | Integration | E2E |
|----------|------------|-------------|-----|
| [Workflow].xaml | ✓ | ✓ | - |

---

## 5. Configuration

### 5.1 Config.xlsx Structure

**Settings Sheet:**

| Name | Value | Description |
|------|-------|-------------|
| [SettingName] | [Value] | [Purpose] |

**Assets Sheet:**

| Name | Type | Description |
|------|------|-------------|
| [AssetName] | Credential / Text | [Purpose] |

### 5.2 Orchestrator Assets

| Asset Name | Type | Description |
|------------|------|-------------|
| [Name] | Credential | [Purpose] |

### 5.3 Orchestrator Queues

| Queue Name | Purpose | Max Retries |
|------------|---------|-------------|
| [Name] | [Purpose] | [Number] |

---

## 6. Integrations

### 6.1 Applications

| Application | Type | Access Method | Authentication |
|-------------|------|---------------|----------------|
| [App Name] | Web / Desktop / API | UI / API | [Auth type] |

### 6.2 APIs

| System | Method | Endpoint | Authentication |
|--------|--------|----------|----------------|
| [Name] | GET/POST | [URL pattern] | [OAuth/API Key/Basic] |

### 6.3 Databases

| Database | Type | Purpose | Access |
|----------|------|---------|--------|
| [Name] | Microsoft SQL Server | [Purpose] | [Read/Write] |

---

## 7. AI Agent Prompt Framework

> This section defines the prompt scaffolding for AI agents used in this project. The Spec Agent references this section when generating new agent prompts to ensure consistency with established project patterns.

### 7.1 Agent Inventory

| Agent Name | Purpose | Platform | Invoked By |
|------------|---------|----------|------------|
| [AgentName] | [What the agent does] | UiPath Agent Builder | [Workflow or trigger] |

### 7.2 Prompt Scaffolding

> All AI agent system prompts in this project follow this structure. When the Spec Agent creates a new agent prompt, it must conform to this scaffolding.

#### Persona

| Property | Value |
|----------|-------|
| **Role** | [Job title / expertise area] |
| **Experience Level** | [Years / seniority context] |
| **Tone** | [Professional / concise / analytical] |

#### Context Block

Define what the agent receives at runtime:

```
You are a [Persona.Role] with [Persona.Experience].

CONTEXT:
You receive [describe input source and trigger condition].
[Additional background the agent needs to understand its operating environment.]
```

#### Task Definition

Numbered steps the agent must perform on every invocation:

```
YOUR TASK:
1. [First analysis step]
2. [Classification or categorization step]
3. [Decision or recommendation step]
4. [Evidence/reasoning step]
5. [Confidence assessment step]
```

#### Decision Framework

Per-scenario logic with specific criteria and thresholds:

| Scenario | Criteria | Recommended Action | Confidence Range |
|----------|----------|--------------------|------------------|
| [Scenario 1] | [Specific conditions / thresholds] | [Action] | [Expected range] |
| [Scenario 2] | [Specific conditions / thresholds] | [Action] | [Expected range] |

> Include which tools the agent should use for each scenario.

#### Guardrails

**Hard Limits** (enforced by both agent prompt and RPA workflow):

| Rule | Rationale |
|------|-----------|
| [Rule description with specific threshold] | [Why this limit exists] |

**Soft Limits** (agent should respect, RPA enforces as backup):

| Rule | Rationale |
|------|-----------|
| [Rule description] | [Why this guideline exists] |

#### Escalation Paths

> Escalations use UiPath Action Center to hand off decisions to humans. Agents decide when to escalate based on prompt instructions. Resolutions feed back into Agent Memory for continuous learning.

| Trigger | Escalation | Action App | Assignee | Expected Outcome |
|---------|-----------|------------|----------|-----------------|
| [When this condition is met] | [Escalation name] | [Action App name] | [Role / person] | [What the human decides] |

**Escalation Configuration:**

| Property | Value |
|----------|-------|
| **Agent Memory Enabled** | [Yes / No — store escalation resolutions for future auto-resolution] |
| **Suspension Behavior** | [Agent suspends until human responds] |
| **Timeout** | [Max wait time before fallback action] |

#### Confidence Scoring

| Range | Evidence Level | Expected Accuracy |
|-------|---------------|-------------------|
| 0.90–1.0 | [What constitutes strong evidence] | [Target %] |
| 0.75–0.89 | [What constitutes good evidence] | [Target %] |
| 0.50–0.74 | [What constitutes weak evidence] | [Target %] |
| 0.0–0.49 | [Insufficient evidence] | Route to manual review |

#### Output Format

```json
{
  "field_1": "description and allowed values",
  "field_2": "description",
  "confidence": 0.00,
  "reasoning": "Required: specific evidence, not generic responses"
}
```

### 7.3 Agent Tools

> Map each tool to its UiPath Agent Builder type. See [Agent Builder tool types](#agent-builder-tool-types) for reference.

| Tool Name | Builder Type | Description | Input | Output |
|-----------|-------------|-------------|-------|--------|
| [ToolName] | [Type] | [What it does] | [Key parameters] | [What it returns] |

**Agent Builder Tool Types:**

| Type | Description | When to Use |
|------|-------------|-------------|
| **RPA workflow** | Automated process connecting multiple steps | Multi-step operations involving UI automation or complex orchestration |
| **API workflow** | Lightweight workflow exposed as API endpoint | Direct API calls to external systems (SAP, QuickBooks, etc.) |
| **Agent** | Another agent performing a smaller job | Delegating a subtask to a specialized sub-agent |
| **Activity** | Single activity (e.g., sending an email) | Simple one-step operations |
| **MCP server** | Exchange context between agents and external tools | Connecting to external tool ecosystems via Model Context Protocol |

**Built-in Tools Available:**

| Tool | Description | When to Use |
|------|-------------|-------------|
| **Analyze Files** | Analyze files with an LLM | Processing documents, PDFs, images attached to agent context |
| **Batch Transform** | Process and transform CSV files | Bulk data processing and transformation |
| **DeepRAG** | Comprehensive synthesis across knowledge sources | Complex queries requiring multi-source knowledge base retrieval |
| **Web Search** | Search the public web | Finding external information, market data, vendor lookups |
| **Web Reader** | Extract readable text from public URLs | Reading specific web pages for reference data |
| **Web Summary** | Generate AI-based summaries of web content | Summarizing external content for context |

### 7.4 Knowledge Base

| Source | Format | Purpose | Update Frequency |
|--------|--------|---------|-----------------|
| [Data source name] | [CSV / Excel / PDF / API] | [What patterns it enables] | [Daily / Weekly / Monthly] |

**Ingestion Configuration:**

| Property | Value |
|----------|-------|
| **Platform** | UiPath AI Center Knowledge Base |
| **Chunk Size** | [tokens] |
| **Embedding Model** | [model name] |

---

## 8. Patterns & Standards

### 8.1 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Arguments | in_, out_, io_ prefix | in_FilePath |
| Variables | Type prefix | str_Status, dt_Data |
| Workflows | PascalCase | 01_ExtractInvoiceData.xaml |

### 8.2 Error Handling

| Exception Type | When to Use | Handling |
|----------------|-------------|----------|
| BusinessRuleException | Business logic violations, data quality issues (missing/incomplete/invalid data), out-of-scope transactions, unauthorized requests, business rule violations | Log as Business Exception, mark for manual review, DO NOT retry (requires human intervention or data correction) |
| ApplicationException | Technical/system errors (app not responding, network timeout, selector not found) | Log as Application Exception, automatic retry if configured, then escalate if retries exhausted |

### 8.3 Logging

| Level | When to Use |
|-------|-------------|
| Info | Major steps, results |
| Error | Exceptions, failures |

---

## 9. Deployment

### 9.1 Environment Configuration

| Environment | Orchestrator Folder |
|-------------|---------------------|
| Dev | /Dev/[Project] |
| Test | /Test/[Project] |
| Prod | /Prod/[Project] |

### 9.2 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| UiPath.System.Activities | [Version] | Core activities |
| [Package] | [Version] | [Purpose] |

---

## 10. Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| [Date] | 1.0 | [Name] | Initial creation |

---

## Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| [Term] | [Definition] |
