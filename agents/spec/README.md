# Spec Agent

Conversational agent that transforms requirements into detailed implementation plans.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | Requirements.md, TDD.md (project context), Coding Standards |
| **Output** | Plan.md, TestScenarios.md |

## What It Does

1. Reads project context (TDD.md) and requirements
2. Analyzes complexity and determines architecture approach
3. Breaks work into atomic workflows
4. Clarifies technical decisions with engineer
5. Generates implementation plan with Autopilot-ready prompts
6. Creates test scenario definitions

## Building the Agent

### 1. Create Agent Project

```
UiPath Studio → New Project → Agent
Name: Spec_Planning_Agent
Type: Conversational
```

### 2. Configure Settings

| Setting | Value |
|---------|-------|
| Agent Type | Conversational |
| Interaction Mode | Synchronous |
| Temperature | 0.3 (more deterministic) |

### 3. Add System Prompt

Copy the system prompt from [PROMPTS.md](./PROMPTS.md).

Key persona: **Solution Architect** with 10+ years UiPath experience.

Core objectives:
- Design optimal workflow architecture
- Break down into atomic, testable workflows
- Specify arguments, variables, activities
- Create comprehensive test scenarios
- Apply coding standards

### 4. Define Tools

| Tool | Purpose |
|------|---------|
| Architecture Analyzer | Determine pattern (REFramework, Dispatcher-Performer, Custom) |
| Workflow Decomposer | Break process into atomic workflows |
| Activity Mapper | Map steps to Modern Design activities |
| Test Scenario Generator | Create unit/integration/E2E tests |

### 5. Planning Flow

```
Load Inputs
├── Read Requirements.md
├── Read TDD.md (project context)
└── Load Coding Standards

Analyze & Design
├── Determine architecture pattern
├── Identify reusable components
└── Plan error handling strategy

Clarify (conversation)
├── Discuss approach with engineer
├── Resolve technical ambiguities
└── Confirm decisions

Decompose
├── Break into workflows
├── Define responsibilities
└── Determine relationships

Specify Details
├── Define arguments/variables
├── Map activities
└── Design error handling

Generate Tests
├── Unit test scenarios
├── Integration test scenarios
└── E2E test scenarios

Output
├── Plan.md
└── TestScenarios.md
```

## Architecture Decision Logic

**Use REFramework when:**
- Processing multiple transactions
- Need retry logic and transaction boundaries
- Standard business process pattern

**Use Dispatcher-Performer when:**
- High volume transactions
- Need parallel processing
- Separate data collection from processing

**Use Custom when:**
- Simple linear process
- Single transaction per run
- API orchestration focused

## Output: Plan.md

Contains:
- Architecture design with rationale
- Workflow breakdown with dependencies
- Detailed specifications per workflow:
  - Arguments (In/Out/InOut with types)
  - Variables
  - Activity-level steps (Autopilot-ready)
  - Error handling
- Data structures and schemas
- Integration specifications
- Test strategy reference

See [examples/output_plan.md](./examples/output_plan.md) for a sample.

## Output: TestScenarios.md

Contains:
- Unit tests (per atomic workflow)
- Integration tests (workflow combinations)
- E2E tests (business scenarios)
- Negative/error tests
- Performance tests
- Mock data requirements

## Deployment

1. Publish to Orchestrator
2. Deploy to UiPath Apps
3. Configure storage for outputs

## Examples

- [Example Plan](./examples/output_plan.md) - Sample Plan.md output

## Related

- [PROMPTS.md](./PROMPTS.md) - Full system prompts and templates
- [Architecture](../../ARCHITECTURE.md) - Overall system design
- [TDD Template](../../templates/TDD_TEMPLATE.md) - Project context template
