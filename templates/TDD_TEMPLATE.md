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

## 7. Patterns & Standards

### 7.1 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Arguments | in_, out_, io_ prefix | in_FilePath |
| Variables | Type prefix | str_Status, dt_Data |
| Workflows | PascalCase | 01_ExtractInvoiceData.xaml |

### 7.2 Error Handling

| Exception Type | When to Use | Handling |
|----------------|-------------|----------|
| BusinessRuleException | Business logic violations, data quality issues (missing/incomplete/invalid data), out-of-scope transactions, unauthorized requests, business rule violations | Log as Business Exception, mark for manual review, DO NOT retry (requires human intervention or data correction) |
| ApplicationException | Technical/system errors (app not responding, network timeout, selector not found) | Log as Application Exception, automatic retry if configured, then escalate if retries exhausted |

### 7.3 Logging

| Level | When to Use |
|-------|-------------|
| Info | Major steps, results |
| Error | Exceptions, failures |

---

## 8. Deployment

### 8.1 Environment Configuration

| Environment | Orchestrator Folder |
|-------------|---------------------|
| Dev | /Dev/[Project] |
| Test | /Test/[Project] |
| Prod | /Prod/[Project] |

### 8.2 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| UiPath.System.Activities | [Version] | Core activities |
| [Package] | [Version] | [Purpose] |

---

## 9. Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| [Date] | 1.0 | [Name] | Initial creation |

---

## Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| [Term] | [Definition] |
