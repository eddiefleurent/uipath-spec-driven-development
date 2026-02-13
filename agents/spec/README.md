# Spec Agent

Conversational agent that transforms requirements into detailed implementation plans.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | Requirements.md, TDD.md (project context, incl. AI agent prompt framework), Coding Standards |
| **Output** | Plan.md (incl. Autopilot prompts + AI agent configs), TestScenarios.md |
| **Persona** | Solution Architect with 10+ years UiPath experience |

## What It Does

1. Reads Requirements.md (which contains relevant business context extracted from PDD)
2. Reads TDD.md for existing automation architecture context and AI agent prompt framework
3. Analyzes complexity and determines architecture approach
4. Breaks work into atomic workflows
5. Clarifies technical decisions with engineer
6. Generates implementation plan with Autopilot-ready prompts
7. **Generates AI agent prompts** following the prompt framework from TDD.md (when project uses agents)
8. Creates comprehensive test scenario definitions

**Note:** The Spec Agent receives business process context via Requirements.md. The Interview Agent extracts relevant PDD content and includes it in Requirements.md, so the Spec Agent does not read PDD.md directly.

**AI Agent Prompt Framework:** For projects that use AI agents built in UiPath Agent Builder, the Spec Agent reads the prompt framework from TDD.md (Section 7) and generates agent prompts following the project's established scaffolding (persona, decision framework, guardrails, tools, confidence scoring, output format, knowledge base).

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
| Temperature | 0.3-0.4 (deterministic for architecture decisions) |

### 3. Add System Prompt

Copy the system prompt from [PROMPTS.md](./PROMPTS.md).

Key persona: **Solution Architect** with 10+ years UiPath experience.

Core objectives:
- Design optimal workflow architecture
- Break down into atomic, testable workflows
- Specify arguments, variables, activities
- Create Autopilot-ready prompts for RPA workflows
- Generate AI agent prompts using project's prompt framework (when applicable)
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
  - Activity-level steps (Autopilot-ready prompts)
  - Error handling
- **AI Agent Configuration** (when applicable):
  - System prompts following project's prompt framework
  - Custom tools definition
  - Knowledge base structure
  - Guardrails and confidence thresholds
- Data structures and schemas
- Integration specifications
- Configuration changes (Config.xlsx, Orchestrator assets)
- Deployment plan
- Test strategy reference

See the [AI Agent Enhancement Example](../../examples/ai-agent-enhancement/04-plan.md) for a comprehensive sample showing both RPA workflow specs and AI agent configuration.

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

See the complete end-to-end example:
- [AI Agent Enhancement Example](../../examples/ai-agent-enhancement/) - Full lifecycle including Spec Agent session
- [03-spec-conversation.md](../../examples/ai-agent-enhancement/03-spec-conversation.md) - Sample multi-turn planning conversation
- [04-plan.md](../../examples/ai-agent-enhancement/04-plan.md) - Comprehensive Plan.md with Autopilot prompts AND AI agent configuration

## Related

- [PROMPTS.md](./PROMPTS.md) - Full system prompts and templates
- [ARCHITECTURE.md](../../ARCHITECTURE.md) - Overall system design
- [Main README](../../README.md) - Project overview and quick start
- [TDD Template](../../templates/TDD_TEMPLATE.md) - Project context template (includes AI agent prompt framework)
- [TDD Example](../../templates/TDD_EXAMPLE.md) - Sample TDD with AI agent prompt framework
- [Autopilot Guide](../../studio/AUTOPILOT_GUIDE.md) - Using Autopilot with Plan.md specs
