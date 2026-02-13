# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation repository** for three AI agents built with UiPath Agent Builder that automate the requirements-to-implementation pipeline for UiPath workflow development. There is **no executable code** here—only documentation, templates, and prompts.

The system helps engineers go from user story to UiPath workflows through:
1. **Interview Agent** - Gathers requirements from user stories
2. **Spec Agent** - Creates implementation plans with Autopilot-ready prompts
3. **TDD Agent** - Updates technical documentation based on workflow changes

## Key Concept: Project Context

These agents work on **individual stories within existing UiPath projects**—not greenfield solutions. The TDD.md file (Technical Design Document, not Test-Driven Development) provides essential context about existing workflows, architecture patterns, coding standards, and integrations. Without this context, agents might suggest incompatible approaches.

## Architecture Overview

### Three-Agent Pipeline

| Agent | Type | Input | Output | Persona |
|-------|------|-------|--------|---------|
| Interview Agent | Conversational | User Story + PDD.md (required) + TDD.md (optional) | Requirements.md | BA Expert |
| Spec Agent | Conversational | Requirements.md + TDD.md + Coding Standards | Plan.md + TestScenarios.md | Solution Architect |
| TDD Agent | Autonomous | Git diff + Plan.md + existing TDD.md | Updated TDD.md | Technical Writer |

### Process Flow

```
PDD.md (required) ─────┐
                       │
User Story ────────────┼──→ Interview Agent → Requirements.md
                       │                           ↓
TDD.md (optional) ─────┘                     Spec Agent → Plan.md + TestScenarios.md
                                                  ↓
                                    Engineer uses UiPath Studio + Autopilot
                                                  ↓
                                         Workflows (.xaml) → GitLab
                                                  ↓
                                         TDD Agent → Updated TDD.md
```

### PDD vs TDD

| Aspect | PDD.md | TDD.md |
|--------|--------|--------|
| **Focus** | AS-IS business process | Automation architecture |
| **Content** | Process steps, business rules, stakeholders | Workflows, activities, integrations, AI agent prompt framework |
| **Created By** | Business Analyst (Task Capture recommended, or process workshops/interviews) | TDD Agent (maintained) |
| **Required** | Yes (for Interview Agent) | Optional but recommended (for existing projects) |

### Architecture Patterns

The Spec Agent selects patterns based on requirements:

- **REFramework**: Multiple independent transactions, retry logic, queue-based processing
- **Dispatcher-Performer**: High volume (>100/day), parallel processing, separable collection/processing
- **Linear/Custom**: Simple sequential process, single transaction, API orchestration

## Documentation Structure

### Core Documentation
- `README.md` - Project overview and quick start
- `ARCHITECTURE.md` - Detailed technical reference for the three-agent system
- `architecture.puml` / `architecture.png` - High-level architecture diagram
- `sequence.png` - Process flow diagram

### Agent Documentation
Each agent has its own directory under `agents/` with:
- `README.md` - Setup guide for building the agent in UiPath Agent Builder
- `PROMPTS.md` - System prompts and templates to configure the agent

### Examples
- `examples/ai-agent-enhancement/` - Complete end-to-end example showing how to enhance an existing automation with AI-powered exception analysis (includes all artifacts from user story through TDD update)

### Templates & Guides
- `templates/PDD_EXAMPLE.md` - Sample Process Definition Document
- `templates/TDD_TEMPLATE.md` - Template for documenting UiPath projects
- `templates/TDD_EXAMPLE.md` - Sample Technical Design Document (result of TDD Agent processing)
- `studio/AUTOPILOT_GUIDE.md` - Guide for using UiPath Autopilot with Plan.md specs

## Key Artifacts (Not in This Repo)

These are created by engineers using the agents:

| Artifact | Created By | Used By |
|----------|------------|---------|
| PDD.md | Business Analyst (Task Capture recommended, or process workshops/interviews) | Interview Agent |
| TDD.md | TDD Agent (maintained) | Interview Agent, Spec Agent |
| Requirements.md | Interview Agent | Spec Agent, Engineer |
| Plan.md | Spec Agent | Engineer (Autopilot), TDD Agent |
| TestScenarios.md | Spec Agent | Engineer |
| Workflows (.xaml) | Engineer + Autopilot | TDD Agent, GitLab |
| AI Agents | Engineer + Agent Builder (using prompts from Plan.md) | UiPath workflows |

## Common Tasks

### Updating Agent Prompts

When modifying prompts in `agents/*/PROMPTS.md`:
1. Ensure consistency with the agent's persona (BA Expert, Solution Architect, Technical Writer)
2. Maintain the multi-turn conversation flow for conversational agents
3. Keep prompts focused on UiPath Modern Design activities
4. Reference the TDD.md context appropriately

### Updating Documentation

When updating README or ARCHITECTURE files:
1. Keep the three-agent pipeline flow clear
2. Maintain consistency between overview (README) and detailed reference (ARCHITECTURE)
3. Update diagrams if process flow changes
4. Ensure examples reflect current output format

### Modifying Templates

When updating `templates/TDD_TEMPLATE.md`:
1. This is read by Interview and Spec agents as project context
2. Changes affect how agents understand existing projects
3. Keep the structure consistent with agent expectations
4. Sections: Project Overview, Architecture, Workflows, Tests, Configuration, Integrations, AI Agent Prompt Framework, Patterns & Standards, Change Log

**AI Agent Prompt Framework (Section 7):**
- For projects using AI agents built in UiPath Agent Builder
- Defines standardized scaffolding for all agent prompts in the project
- Elements: Persona, Context Block, Task Definition, Decision Framework, Guardrails, Confidence Scoring, Output Format, Tools, Knowledge Base
- Spec Agent reads this framework and generates new agent prompts following the project's established patterns
- See `templates/TDD_TEMPLATE.md` (Section 7) for scaffolding structure
- See `templates/TDD_EXAMPLE.md` (Section 7) for a filled-in example

## UiPath-Specific Context

### Modern Design Activities
Agents recommend UiPath Modern Design activities (not classic):
- Use Application/Browser (not Open Browser)
- Type Into, Click, Get Text (Modern variants)
- HTTP Request for API calls
- Document Understanding for PDF extraction
- Data Extraction Scope for structured data

### Error Handling Patterns
- **BusinessRuleException**: Business logic violations, data quality issues → Log and mark for review, DO NOT retry
- **ApplicationException**: Technical/system errors → Log and retry if configured

### Naming Conventions
- Arguments: `in_`, `out_`, `io_` prefix
- Variables: Type prefix (`str_`, `dt_`, `bool_`)
- Workflows: PascalCase with optional numeric prefix

## Working with This Repository

### No Build/Test Commands
This is a documentation-only repository. There are no build steps, tests to run, or code to compile.

### Diagram Editing
- Source: `architecture.puml` (PlantUML format)
- Outputs: `architecture.png`, `sequence.png`
- To regenerate diagrams, use PlantUML tools with the .puml source

### Git Commands for Documentation Updates

Get workflow changes (used by TDD Agent):
```bash
# For last commit
git diff HEAD~1 -- "*.xaml"

# For a PR/branch
git diff origin/main...HEAD -- "*.xaml"

# Just file names
git diff HEAD~1 --name-only -- "*.xaml"
```

## UiPath Autopilot Integration

The Spec Agent creates "Autopilot-ready" prompts in Plan.md. Engineers paste these into UiPath Studio Autopilot to generate workflows. Key points:

- Prompts must be specific about argument types, Modern Design activities, and error handling
- Engineers review and refine generated workflows
- See `studio/AUTOPILOT_GUIDE.md` for detailed workflow

## Important Notes

1. **No Code Here**: This repo contains documentation, prompts, and templates only
2. **Agent Builder Setup**: The three system agents (Interview, Spec, TDD) are built in UiPath Agent Builder (separate platform)
3. **Studio + Autopilot**: Actual workflow implementation happens in UiPath Studio
4. **TDD.md is Critical**: Without project context (TDD.md), agents cannot make appropriate recommendations
5. **This is Not TDD**: TDD.md = Technical Design Document, not Test-Driven Development
6. **PDD Creation Flexibility**: UiPath Task Capture is recommended but not required—PDDs can be created through process workshops, interviews, or other documentation methods
7. **AI Agent Prompt Framework**: For projects using AI agents, TDD.md Section 7 defines prompt scaffolding that ensures consistent agent design across the project
8. **Complete Example Available**: See `examples/ai-agent-enhancement/` for end-to-end lifecycle demonstration
