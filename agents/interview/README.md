# Interview Agent

Conversational agent that gathers requirements from user stories through multi-turn chat.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | User Story, TDD.md (project context) |
| **Output** | Requirements.md |

## What It Does

1. Reads project context (TDD.md) to understand existing architecture
2. Analyzes the user story to identify gaps
3. Asks clarifying questions in logical groups
4. Validates understanding with the engineer
5. Generates structured Requirements.md

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
├── Read TDD.md (project context)
└── Analyze gaps

Interview Loop
├── Ask question group
├── Capture response
├── Analyze for follow-ups
└── Check completion

Validate
├── Summarize understanding
├── Confirm with engineer
└── Address corrections

Generate
└── Output Requirements.md
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
