# Spec-Driven Development for UiPath

AI-powered agents that help engineers go from user story to UiPath workflows. Write the spec, let AI generate the implementation.

## The Problem

Turning user stories into production-ready UiPath workflows involves:
- Gathering detailed requirements (often incomplete in tickets)
- Designing the right architecture and workflow structure
- Writing workflow specifications detailed enough for implementation
- Keeping documentation in sync with code changes

This is time-consuming and error-prone, especially for complex automations.

## The Solution

Three conversational agents built with UiPath Agent Builder that automate the requirements-to-implementation pipeline:

| Agent | Role | Input | Output |
|-------|------|-------|--------|
| **Interview Agent** | BA Expert | User Story | Requirements.md |
| **Spec Agent** | Solution Architect | Requirements.md | Plan.md + TestScenarios.md |
| **TDD Agent** | Technical Writer | Git diff + Plan.md | Updated TDD.md |

Engineers use **UiPath Studio + Autopilot** to implement workflows from Plan.md.

## Architecture

![High-Level Architecture](./architecture.png)

## Process Flow

![Process Flow](./sequence.png)

## How It Works

### 1. Requirements Gathering
Engineer provides a user story to the **Interview Agent**, which:
- Reads project context (TDD.md) to understand existing architecture
- Asks clarifying questions through multi-turn conversation
- Generates comprehensive Requirements.md

### 2. Planning
The **Spec Agent** takes approved requirements and:
- Determines optimal architecture (REFramework, Dispatcher-Performer, etc.)
- Decomposes into atomic, testable workflows
- Creates detailed Plan.md with Autopilot-ready prompts
- Generates TestScenarios.md for testing

### 3. Implementation
Engineer uses **UiPath Studio + Autopilot**:
- Open Plan.md workflow specifications
- Paste specs into Autopilot to generate workflows
- Review, refine, and commit to GitLab

### 4. Documentation Update
The **TDD Agent** receives git diff and:
- Cross-references changes with Plan.md
- Updates TDD.md with new/changed workflow documentation
- Maintains the project's single source of truth

## Key Concept: Project Context

These agents work on **individual stories** within existing projects—not greenfield solutions. The TDD.md file (Technical Design Document—not to be confused with Test-Driven Development) provides essential context:

- Existing workflows and their responsibilities
- Architecture patterns in use
- Coding standards and conventions
- Integration details

Without this context, agents might suggest incompatible approaches. With it, they can say "modify existing X" vs "create new Y".

## Quick Start

1. **Set up project documentation**
   - Create TDD.md using the [template](./templates/TDD_TEMPLATE.md)
   - Document your existing project architecture

2. **Build the Interview Agent**
   - Follow [agents/interview/README.md](./agents/interview/README.md)
   - Use prompts from [agents/interview/PROMPTS.md](./agents/interview/PROMPTS.md)

3. **Build the Spec Agent**
   - Follow [agents/spec/README.md](./agents/spec/README.md)
   - Use prompts from [agents/spec/PROMPTS.md](./agents/spec/PROMPTS.md)

4. **Test with a real user story**
   - Start with a well-understood story for validation
   - Compare agent output to how you'd approach it manually

## Documentation

| Document | Purpose |
|----------|---------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Deep-dive technical reference |
| [Interview Agent](./agents/interview/README.md) | Building the requirements agent |
| [Spec Agent](./agents/spec/README.md) | Building the planning agent |
| [TDD Agent](./agents/tdd/README.md) | Building the documentation agent |
| [Autopilot Guide](./studio/AUTOPILOT_GUIDE.md) | Using Autopilot with Plan.md |
| [TDD Template](./templates/TDD_TEMPLATE.md) | Project documentation template |

## Prerequisites

- UiPath Studio 2024.10+
- UiPath Agent Builder access
- UiPath Autopilot enabled
- GitLab (or other VCS) for version control

## Project Structure

```
uipath-spec-driven-development/
├── README.md                 # This file - start here
├── ARCHITECTURE.md           # Detailed technical reference
├── architecture.puml         # Diagram source (PlantUML)
├── architecture.png          # High-level architecture diagram
├── sequence.png              # Process flow diagram
│
├── agents/
│   ├── interview/            # Interview Agent
│   │   ├── README.md         # Setup guide
│   │   ├── PROMPTS.md        # System prompts
│   │   └── examples/         # Sample chat and output
│   ├── spec/                 # Spec Agent
│   │   ├── README.md         # Setup guide
│   │   ├── PROMPTS.md        # System prompts
│   │   └── examples/         # Sample plan output
│   └── tdd/                  # TDD Agent
│       ├── README.md         # Setup guide
│       └── PROMPTS.md        # System prompts
│
├── templates/
│   └── TDD_TEMPLATE.md       # Project documentation template
│
└── studio/
    └── AUTOPILOT_GUIDE.md    # Using Autopilot with Plan.md
```

## Artifacts

| Artifact | Created By | Used By |
|----------|------------|---------|
| TDD.md | TDD Agent (maintained) | Interview Agent, Spec Agent |
| Requirements.md | Interview Agent | Spec Agent, Engineer |
| Plan.md | Spec Agent | Engineer (Autopilot), TDD Agent |
| TestScenarios.md | Spec Agent | Engineer |
| Workflows (.xaml) | Engineer + Autopilot | TDD Agent, GitLab |
