# SDLC Agents - Architecture Reference

Detailed technical reference for the SDLC Agents system. For an overview, see [README.md](./README.md).

---

## System Components

### Interview Agent

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | User Story, TDD.md (project context) |
| **Output** | Requirements.md |
| **Persona** | BA Expert |

**Responsibilities:**
- Read project context to understand existing architecture
- Analyze user story to identify gaps and ambiguities
- Conduct multi-turn conversation to gather requirements
- Generate structured Requirements.md

**Tools:**
| Tool | Purpose |
|------|---------|
| Question Generator | Creates targeted questions based on story analysis |
| Gap Analyzer | Identifies missing information |
| Doc Generator | Produces Requirements.md |

**Documentation:** [agents/interview/](./agents/interview/)

---

### Spec Agent

| Property | Value |
|----------|-------|
| **Type** | Conversational |
| **Platform** | UiPath Agent Builder |
| **UI** | UiPath Apps |
| **Input** | Requirements.md, TDD.md, Coding Standards |
| **Output** | Plan.md, TestScenarios.md |
| **Persona** | Solution Architect |

**Responsibilities:**
- Analyze requirements against existing project architecture
- Determine optimal architecture pattern
- Decompose into atomic, testable workflows
- Specify arguments, variables, activities for each workflow
- Create Autopilot-ready prompts
- Define test scenarios (unit, integration, E2E)

**Tools:**
| Tool | Purpose |
|------|---------|
| Architecture Analyzer | Determine pattern (REFramework, Dispatcher-Performer, Custom) |
| Workflow Decomposer | Break process into atomic workflows |
| Activity Mapper | Map steps to Modern Design activities |
| Test Scenario Generator | Create comprehensive test definitions |

**Documentation:** [agents/spec/](./agents/spec/)

---

### TDD Agent

| Property | Value |
|----------|-------|
| **Type** | Autonomous |
| **Platform** | UiPath Agent Builder |
| **Input** | Git diff, Plan.md, existing TDD.md |
| **Output** | Updated TDD.md |
| **Persona** | Technical Writer |

**Responsibilities:**
- Parse git diff to identify workflow changes
- Cross-reference with Plan.md to understand intent
- Update TDD.md with new/modified/deleted workflows
- Maintain documentation accuracy and consistency

**Tools:**
| Tool | Purpose |
|------|---------|
| Diff Parser | Extract workflow changes from git diff |
| TDD Updater | Merge changes into existing TDD.md |
| Doc Generator | Format updated TDD.md |

**Documentation:** [agents/tdd/](./agents/tdd/)

---

### UiPath Studio + Autopilot

**Not an agent**—this is the engineer using Studio to implement workflows.

| Property | Value |
|----------|-------|
| **Input** | Plan.md, TestScenarios.md |
| **Output** | Workflow files (.xaml) |
| **Key Features** | Text-to-Workflow, Text-to-Code, Expression Generation |

**Workflow:**
1. Engineer opens Plan.md
2. Copies workflow specification into Autopilot
3. Autopilot generates workflow
4. Engineer reviews, refines, commits to GitLab

**Documentation:** [studio/AUTOPILOT_GUIDE.md](./studio/AUTOPILOT_GUIDE.md)

---

## The TDD.md File

The Technical Design Document is the **single source of truth** for project architecture. It enables agents to understand context and make appropriate recommendations.

### Contents

| Section | Purpose |
|---------|---------|
| Project Overview | What the automation does, business process |
| Architecture | Pattern used, component diagram |
| Workflows | Complete inventory with full specifications |
| Tests | Test inventory and coverage |
| Configuration | Config.xlsx structure, assets, queues |
| Integrations | Systems, APIs, credentials |
| Patterns & Standards | Error handling, logging, naming conventions |
| Change Log | History of modifications |

### Why It Matters

| Without TDD.md | With TDD.md |
|----------------|-------------|
| Agents suggest generic approaches | Agents align with existing patterns |
| May recommend incompatible solutions | Can say "modify X" vs "create Y" |
| Duplicates existing functionality | Reuses established components |
| Inconsistent with project conventions | Follows established standards |

### Maintenance

- **Updated by:** TDD Agent after workflow changes
- **Triggered by:** Engineer providing git diff + Plan.md + existing TDD.md
- **Template:** [templates/TDD_TEMPLATE.md](./templates/TDD_TEMPLATE.md)

---

## Architecture Patterns

The Spec Agent selects patterns based on requirements:

### REFramework

**Use when:**
- Processing multiple independent transactions
- Need retry logic at transaction level
- Standard queue-based processing
- Require transaction status tracking

**Example:** Invoice processing from Orchestrator queue

### Dispatcher-Performer

**Use when:**
- High transaction volume (>100/day)
- Can benefit from parallel processing
- Data collection is separable from processing
- Need to scale processing independently

**Example:** Large-scale data migration

### Linear/Custom

**Use when:**
- Simple sequential process
- Single transaction per run
- API orchestration focused
- No retry requirements at item level

**Example:** Daily report generation

---

## Process Flow (Detailed)

### 1. Requirements Gathering

```
1. Engineer provides user story to Interview Agent
2. Agent reads TDD.md to understand project context
3. Multi-turn conversation to clarify requirements
   - Business context questions
   - Process flow questions
   - Data & integration questions
   - Error handling questions
   - Technical requirements questions
4. Agent validates understanding with summary
5. Agent generates Requirements.md
6. Engineer reviews and approves
```

### 2. Planning

```
1. Engineer provides Requirements.md to Spec Agent
2. Agent reads Requirements.md + TDD.md
3. Agent determines architecture approach
4. Conversation to clarify technical decisions
5. Agent decomposes into workflows
6. Agent generates Plan.md + TestScenarios.md
7. Engineer reviews and approves
```

### 3. Implementation

```
1. Engineer opens Plan.md in Studio
2. For each workflow in plan:
   a. Copy Autopilot-ready prompt
   b. Paste into Autopilot
   c. Review generated workflow
   d. Refine as needed
   e. Apply team standards
3. Run Workflow Analyzer
4. Commit workflows to GitLab
```

### 4. Testing

```
1. Engineer opens TestScenarios.md
2. For each test scenario:
   a. Use Autopilot to generate test workflow
   b. Configure test data
   c. Run tests
3. Fix any issues found
4. Commit test workflows to GitLab
```

### 5. Documentation Update

```
1. Engineer generates git diff:
   git diff origin/main...HEAD -- "*.xaml"
2. Engineer provides to TDD Agent:
   - Git diff
   - Plan.md
   - Existing TDD.md
3. TDD Agent outputs updated TDD.md
4. Engineer reviews and commits
```

---

## Artifact Flow

```
User Story
    │
    ▼
┌─────────────────┐
│ Interview Agent │
└────────┬────────┘
         │ Requirements.md
         ▼
┌─────────────────┐
│   Spec Agent    │
└────────┬────────┘
         │ Plan.md + TestScenarios.md
         ▼
┌─────────────────┐
│ Studio/Autopilot│
└────────┬────────┘
         │ Workflows (.xaml)
         ▼
┌─────────────────┐
│     GitLab      │
└────────┬────────┘
         │ git diff
         ▼
┌─────────────────┐
│   TDD Agent     │
└────────┬────────┘
         │ Updated TDD.md
         ▼
      GitLab
```

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| Agent Platform | UiPath Agent Builder |
| Agent UI | UiPath Apps |
| Workflow Generation | UiPath Autopilot |
| Workflow Design | Modern Design Activities |
| Version Control | GitLab |

---

## Prerequisites

| Requirement | Version/Notes |
|-------------|---------------|
| UiPath Studio | 2024.10+ |
| Agent Builder | Access required |
| Autopilot | Enabled in Studio |
| GitLab | Or other VCS |
| Coding Standards | Documented for project |

---

## Related Documents

| Document | Purpose |
|----------|---------|
| [README.md](./README.md) | Project overview and quick start |
| [Interview Agent](./agents/interview/README.md) | Setup and usage |
| [Spec Agent](./agents/spec/README.md) | Setup and usage |
| [TDD Agent](./agents/tdd/README.md) | Setup and usage |
| [TDD Template](./templates/TDD_TEMPLATE.md) | Project documentation template |
| [Autopilot Guide](./studio/AUTOPILOT_GUIDE.md) | Using Autopilot with Plan.md |
