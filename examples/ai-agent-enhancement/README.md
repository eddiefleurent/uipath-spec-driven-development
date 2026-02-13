# Complete Example: AI Agent Enhancement

This directory contains a **complete end-to-end example** showing how the three-agent system works to enhance an existing automation with AI-powered exception analysis.

## Scenario

**Starting Point:** Invoice Approval automation v1.0 (production, no AI)
**Enhancement:** Add AI agent to analyze validation exceptions and auto-resolve common issues
**Goal:** Reduce manual review queue from 30% to ≤20% of invoices

---

## The Complete Lifecycle

Follow the artifacts in order to see the full flow:

### 0. User Story
📄 **[00-user-story.md](./00-user-story.md)**

Product owner (Jane Smith, AP Manager) submits enhancement request:
- **Problem:** 30% of invoices require manual review
- **Insight:** ~40% of exceptions are routine and could be auto-resolved
- **Goal:** Reduce manual review by 30% using AI agent

---

### 1. Interview Agent Session
📄 **[01-interview-conversation.md](./01-interview-conversation.md)**

Engineer provides user story to Interview Agent, which:
- Reads PDD.md (Invoice Approval Process - AS-IS business process)
- Reads TDD.md v1.0 (current automation architecture)
- Asks 8 clarifying questions to fill gaps not covered by PDD/TDD
- Extracts ONLY relevant context for this specific enhancement

**Key Learning:** Interview Agent doesn't ask redundant questions. It uses PDD/TDD context to focus on what's missing.

---

### 2. Requirements Output
📄 **[02-requirements.md](./02-requirements.md)**

Interview Agent generates focused requirements including:
- 7 functional requirements (FR-1 through FR-7)
- 4 non-functional requirements (performance, accuracy, security)
- Integration points with existing SAP/QuickBooks APIs
- Rollout strategy (shadow mode → pilot → full)
- Success metrics (≥90% accuracy, <5s response time)

**Note:** Requirements reference existing TDD.md v1.0 architecture, ensuring compatibility.

---

### 3. Spec Agent Session
📄 **[03-spec-conversation.md](./03-spec-conversation.md)**

Engineer provides requirements to Spec Agent, which:
- Designs integration point (after ValidateAgainstPO.xaml fails)
- Creates new workflow: `AnalyzeExceptionWithAI.xaml`
- Designs graceful degradation pattern (AI fails → route to manual review)
- Plans **both RPA workflow AND AI agent configuration**

**Key Decision:** AI agent sits between validation failure and exception queue as intelligent filter.

---

### 4. Implementation Plan
📄 **[04-plan.md](./04-plan.md)**

Spec Agent generates comprehensive plan with:

#### Section 2: Autopilot Prompt for AnalyzeExceptionWithAI.xaml
**Detailed, prescriptive prompt** including:
- Exact activity names (Invoke AI Agent, Deserialize JSON, etc.)
- All arguments and variables with types
- Step-by-step logic (45+ steps)
- Try-Catch error handling
- Guardrails implementation

**Why so detailed?** Autopilot generates 90%+ correct code when given specific instructions.

#### Section 3: Autopilot Prompt for Process.xaml Modification
Shows exactly where and how to integrate AI workflow into existing REFramework Process.xaml.

#### Section 4: AI Agent Configuration
**Complete Agent Builder setup:**
- System prompt for InvoiceExceptionAnalyzer agent
- 4 custom tools (CheckVendorAlias, CheckDuplicateInvoice, etc.)
- Knowledge base structure (12 months historical data)
- Guardrails (amount limits, confidence thresholds)
- Agent prompt template

**Key Insight:** Spec Agent creates prompts for BOTH the RPA workflow (Autopilot) AND the AI agent itself (Agent Builder).

#### Sections 5-6: Configuration and Deployment
- Config.xlsx updates (7 new settings)
- Orchestrator assets (AIAgentAPIKey)
- 5-phase rollout plan (Agent setup → Dev → Shadow → Pilot → Prod)

---

### 5. Test Scenarios
📄 **[05-test-scenarios.md](./05-test-scenarios.md)**

Spec Agent generates comprehensive test plan:
- **Unit Tests (6 scenarios):** Workflow logic with mocked AI responses
- **Integration Tests (10 scenarios):** Real AI agent with known test cases
- **E2E Tests (5 scenarios):** Full invoice flow from download to resolution
- **Shadow Mode Validation:** 2-week validation plan with metrics
- **Performance Tests:** Volume, concurrency, stress testing

**Each scenario includes:**
- Test data setup
- Expected AI response
- Expected workflow behavior
- Validation criteria

---

### 6. Autopilot Session
📄 **[06-autopilot-session.md](./06-autopilot-session.md)**

**Step-by-step guide** showing how engineer uses UiPath Autopilot with Plan.md:

1. Open Studio, create `AnalyzeExceptionWithAI.xaml`
2. Open Autopilot panel
3. **Paste complete prompt from Plan.md Section 2**
4. Review generated workflow (arguments, variables, activities)
5. Fine-tune (replace placeholder Invoke AI Agent activity with actual)
6. Test with mock data
7. Troubleshoot common issues

**Time Savings:**
- Manual coding: 2-3 hours
- With Autopilot: 30 minutes (~75% reduction)

**Key Takeaway:** Detailed prompts from Spec Agent enable Autopilot to generate production-quality code on first try.

---

### 7. Git Diff
📄 **[07-git-diff.txt](./07-git-diff.txt)**

Sample git diff showing what engineer commits after implementation:
- New file: `AnalyzeExceptionWithAI.xaml` (+287 lines)
- Modified: `Framework/Process.xaml` (+141 lines, AI integration)
- Modified: `Data/Config.xlsx` (+7 settings)
- Modified: `project.json` (+1 package dependency)

**Total Impact:** +428 lines of code

---

### 8. TDD Agent Update
📄 **[08-tdd-update.md](./08-tdd-update.md)**

Shows how TDD Agent processes git diff to update documentation:

**Input:**
- Git diff (4 files changed)
- Plan.md (for context on why changes were made)

**Processing:**
1. Parse diff to identify new/modified workflows
2. Cross-reference Plan.md to understand purpose
3. Identify 9 TDD.md sections needing updates
4. Generate comprehensive workflow documentation
5. Add AI integration details
6. Update configuration, dependencies, change log

**Output:** TDD.md v2.0 (single source of truth)

**Reference:** [templates/TDD_EXAMPLE.md](../../templates/TDD_EXAMPLE.md) - The complete updated TDD

---

## Key Concepts Demonstrated

### 1. Context Engineering
The agents use structured context (PDD + TDD) to make informed decisions:
- **Interview Agent:** Reads PDD to understand AS-IS process, reads TDD to understand existing automation
- **Spec Agent:** References TDD to ensure compatibility with existing architecture
- **TDD Agent:** Cross-references Plan.md to understand why changes were made

### 2. Autopilot-Ready Prompts
Plan.md prompts are **prescriptive, not conceptual:**
- ❌ "Create a workflow that calls an AI agent" (too vague)
- ✅ "Use Invoke AI Agent activity, set timeout to 5000ms, deserialize JSON response, apply guardrails..." (specific)

Result: Autopilot generates 90%+ correct code

### 3. AI Agent + RPA Integration
Shows how agentic AI enhances traditional RPA:
- **RPA handles:** Structured tasks (download, extract, upload)
- **AI Agent handles:** Judgment calls (is this vendor alias legitimate?)
- **Integration:** Graceful degradation (AI fails → fallback to manual review)

### 4. Incremental Enhancement
This is NOT a greenfield project:
- v1.0 automation already exists and works
- Enhancement adds AI agent as **new component**
- Existing workflows remain unchanged (only Process.xaml modified for integration)
- Fallback behavior = current v1.0 behavior (manual review)

---

## How to Use This Example

### For Learning
1. Read artifacts in order (00 through 08)
2. Notice how context flows from one agent to the next
3. See how detailed prompts enable Autopilot to generate code
4. Understand graceful degradation and guardrails

### For Your Own Projects
1. Adapt user story template (00-user-story.md) for your enhancement
2. Run Interview Agent with your PDD + TDD
3. Run Spec Agent with requirements
4. Use Plan.md prompts in Autopilot to build workflows
5. Run TDD Agent to update documentation

### For AI Agent Projects
Study Section 4 of Plan.md to see:
- How to structure AI agent system prompts
- How to define custom tools for agents
- How to set up knowledge bases
- How to configure guardrails
- How to integrate Agent Builder with UiPath workflows

---

## Artifacts Summary

| # | File | Agent | Purpose |
|---|------|-------|---------|
| 0 | 00-user-story.md | Human | Enhancement request from product owner |
| 1 | 01-interview-conversation.md | Interview | Shows requirement gathering with PDD/TDD context |
| 2 | 02-requirements.md | Interview | Focused requirements output |
| 3 | 03-spec-conversation.md | Spec | Shows solution design process |
| 4 | **04-plan.md** | Spec | **Autopilot prompts + AI agent config** |
| 5 | 05-test-scenarios.md | Spec | Comprehensive test plan |
| 6 | **06-autopilot-session.md** | Human | **How to use Autopilot with Plan.md** |
| 7 | 07-git-diff.txt | Human | Sample workflow changes |
| 8 | 08-tdd-update.md | TDD | Documentation update process |

**Most Important:**
- **04-plan.md** - Contains Autopilot-ready prompts (THE KEY DELIVERABLE)
- **06-autopilot-session.md** - Shows how to use those prompts in Studio

---

## Related Documentation

- [PDD Example](../../templates/PDD_EXAMPLE.md) - Process Definition Document for Invoice Approval
- [TDD Template](../../templates/TDD_TEMPLATE.md) - Empty template for new projects
- [TDD Example v2.0](../../templates/TDD_EXAMPLE.md) - Result after TDD Agent updates (this enhancement)
- [Autopilot Guide](../../studio/AUTOPILOT_GUIDE.md) - General guide for using Autopilot

---

## Questions & Discussion

**Q: Why is the Autopilot prompt so detailed?**
A: Autopilot generates better code with specific instructions. Vague prompts produce generic code that needs heavy editing.

**Q: Can I use this approach without Agent Builder?**
A: Yes! The Interview → Spec → Autopilot → TDD flow works for any enhancement. Agent Builder is only needed if your automation includes an AI agent component.

**Q: Does TDD Agent require Plan.md?**
A: Recommended but not required. TDD Agent can work from git diff alone, but Plan.md provides crucial context about WHY changes were made.

**Q: How long does this process take?**
A: For this enhancement (adding AI agent to existing automation):
- Interview: 30 minutes
- Spec: 45 minutes
- Implementation (with Autopilot): 1-2 hours
- Testing: 4-8 hours (shadow mode)
- Total: ~2 days (vs. 1-2 weeks manual)

**Q: Can I skip shadow mode?**
A: Not recommended for AI agents. Shadow mode validates AI accuracy before enabling auto-resolution. Skipping risks incorrect auto-resolutions in production.

---

## Next Steps

Try this on your own project:
1. Document your AS-IS process (create PDD.md)
2. Document your current automation (create TDD.md)
3. Write an enhancement user story
4. Follow this lifecycle with the agents
5. Use Autopilot to implement
6. Share feedback!

---

**Questions or feedback?** [Open an issue](https://github.com/eddiefleurent/uipath-spec-driven-development/issues)
