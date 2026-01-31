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
| **Input** | User Story, PDD.md (required), TDD.md (optional) |
| **Output** | Requirements.md |
| **Persona** | BA Expert |

**Responsibilities:**
- Read PDD.md to understand AS-IS business process
- Extract ONLY relevant business context for this specific user story
- Read TDD.md (if provided) to understand existing automation architecture
- Analyze user story to identify gaps not covered by PDD/TDD
- Conduct multi-turn conversation to gather requirements (focused on gaps)
- Generate focused Requirements.md with relevant business context

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

## The PDD.md File

The Process Definition Document captures the **AS-IS business process**—what humans do today before automation. It is **required for** the Interview Agent's inputs.

### PDD vs TDD

| Aspect | PDD.md | TDD.md |
|--------|--------|--------|
| **Focus** | Business process documentation | Automation architecture documentation |
| **State** | AS-IS (current manual process) | TO-BE (automation implementation) |
| **Content** | Process steps, business rules, stakeholders, statistics | Workflows, activities, error handling, integrations |
| **Created By** | Business Analyst + UiPath Task Capture | TDD Agent (from git diff + Plan.md) |
| **Created When** | Before automation (during process analysis) | After implementation (from code changes) |
| **Used By** | Interview Agent | Interview Agent, Spec Agent |

### What PDD Contains

Based on UiPath Task Capture documentation, a PDD includes:

| Section | Purpose |
|---------|---------|
| Introduction | Purpose, objectives, process contacts, prerequisites |
| Process Documentation | AS-IS process map, process statistics |
| Detailed Process Steps | Sequences, actions, timeframes, screenshots |
| To-Be Process Analysis | Automation opportunities, scope definition |
| Exception Planning | Business exceptions, application errors, reporting |

### Why PDD Matters

| Without PDD | With PDD |
|-------------|----------|
| Interview Agent asks many basic process questions | Interview Agent focuses on gaps and clarifications |
| Requirements.md lacks business context | Requirements.md includes relevant business rules |
| Spec Agent designs without process understanding | Spec Agent aligns design with business process |
| Longer interview sessions | Shorter, more focused conversations |

### How Interview Agent Uses PDD

**Critical:** The Interview Agent extracts **ONLY relevant context** from PDD into Requirements.md.

- ✓ Include process steps affected by this user story
- ✓ Include applicable business rules and decisions
- ✓ Include relevant stakeholders and exceptions
- ✗ Do NOT copy the entire PDD
- ✗ Do NOT include unrelated process steps

**Template:** [templates/PDD_EXAMPLE.md](./templates/PDD_EXAMPLE.md)

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
2. Agent reads PDD.md to understand AS-IS business process (required)
3. Agent reads TDD.md to understand existing automation (optional)
4. Agent extracts ONLY relevant context from PDD for this story
5. Multi-turn conversation to clarify requirements (focused on gaps)
   - Questions about gaps not covered by PDD/TDD
   - Process flow clarifications
   - Data & integration questions
   - Error handling questions
   - Technical requirements questions
6. Agent validates understanding with summary
7. Agent generates Requirements.md (with relevant PDD context)
8. Engineer reviews and approves
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
PDD.md (required)    User Story
    │                    │
    └────────┬───────────┘
             ▼
    ┌─────────────────┐
    │ Interview Agent │◄── TDD.md (optional)
    └────────┬────────┘
             │ Requirements.md
             ▼
    ┌─────────────────┐
    │   Spec Agent    │◄── TDD.md
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
| [PDD Example](./templates/PDD_EXAMPLE.md) | Sample Process Definition Document |
| [TDD Template](./templates/TDD_TEMPLATE.md) | Project documentation template |
| [Autopilot Guide](./studio/AUTOPILOT_GUIDE.md) | Using Autopilot with Plan.md |
