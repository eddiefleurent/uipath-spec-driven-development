# Interview Agent

Conversational agent that gathers requirements from user stories through multi-turn chat.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | User Story, PDD.md (required), TDD.md (optional) |
| **Output** | Requirements.md |

## What It Does

1. Reads PDD.md (required) to understand the AS-IS business process
2. Reads TDD.md (optional) to understand existing automation architecture
3. Analyzes the user story to identify gaps not covered by PDD/TDD
4. Extracts ONLY relevant business context from PDD for this specific story
5. Asks clarifying questions in logical groups (focused on gaps)
6. Validates understanding with the engineer
7. Generates structured Requirements.md with focused, relevant content

## Key Concept: PDD vs TDD

| Document | Focus | Content |
|----------|-------|---------|
| **PDD.md** (required) | AS-IS business process | Process steps, business rules, stakeholders, decisions |
| **TDD.md** (optional) | Automation architecture | Workflows, activities, error handling, integrations |

The Interview Agent uses PDD to understand *what the business does today* and TDD to understand *what automation already exists*.

## Building the Agent

### 1. Create Agent Project

```
UiPath Studio → New Project → Agent
Name: Interview_Agent
Type: Conversational
```

### 2. Configure Settings

| Setting | Value |
|---------|-------|
| Agent Type | Conversational |
| Interaction Mode | Synchronous |
| Temperature | 0.7 |

### 3. Add System Prompt

Copy the system prompt from [PROMPTS.md](./PROMPTS.md).

Key persona: **BA Expert** with 10+ years RPA experience.

### 4. Define Tools

| Tool | Purpose |
|------|---------|
| Question Generator | Creates targeted questions based on story analysis |
| Gap Analyzer | Identifies missing information |
| Doc Generator | Produces Requirements.md |

### 5. Conversation Flow

```
Initialize
├── Load user story
├── Read PDD.md (business process - required)
├── Read TDD.md (automation context - optional)
├── Extract relevant PDD context for this story
└── Analyze gaps (what's NOT in PDD/TDD)

Interview Loop
├── Ask question group (focused on gaps)
├── Capture response
├── Analyze for follow-ups
└── Check completion

Validate
├── Summarize understanding
├── Confirm with engineer
└── Address corrections

Generate
└── Output Requirements.md (with relevant PDD context)
```

## Question Groups

The agent asks questions in this order:

1. **Business Context** - Problem, stakeholders, success metrics
2. **Process Flow** - Steps, triggers, decision points
3. **Data & Integration** - Sources, formats, systems
4. **Error Handling** - Failures, edge cases, recovery
5. **Technical Requirements** - Performance, security, scheduling

## Output: Requirements.md

Structured document with:
- Business Context
- Functional Requirements
- Data Requirements
- Integration Requirements
- Error Handling & Edge Cases
- Non-Functional Requirements
- Constraints & Assumptions
- Success Criteria

See [examples/example_output.md](./examples/example_output.md) for a sample.

## Deployment

1. Publish to Orchestrator
2. Deploy to UiPath Apps
3. Configure storage location for Requirements.md output

## Examples

- [Example Chat](./examples/example_chat.md) - Sample conversation
- [Example Output](./examples/example_output.md) - Sample Requirements.md

## Related

- [PROMPTS.md](./PROMPTS.md) - Full system prompts and question templates
- [Architecture](../../ARCHITECTURE.md) - Overall system design
