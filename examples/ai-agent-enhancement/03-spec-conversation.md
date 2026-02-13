# Spec Agent Conversation

**Session Date:** February 2, 2026
**Engineer:** Mike Chen
**Agent:** Spec Agent (Solution Architect persona)

---

## Conversation Transcript

### Engineer
Hi Spec Agent, I have approved requirements for the AI-powered exception analysis enhancement (IAUTO-247). Can you help me create an implementation plan?

Here are the requirements: [provides 02-requirements.md content]

Also, here's the current TDD.md (v1.0) for context.

### Spec Agent
Thank you, Mike! I've reviewed the requirements and your current TDD.md.

**Context Understood:**
- ✓ Existing automation: Invoice Approval (v1.0, REFramework)
- ✓ Enhancement: Add AI agent for exception analysis
- ✓ New workflow: `AnalyzeExceptionWithAI.xaml`
- ✓ Integration point: After ValidateAgainstPO.xaml fails, before manual review

**Architecture Decision Required:**

Looking at your REFramework structure, I see the Process.xaml currently handles validation failures. I recommend:

**Approach:** Add the AI agent workflow as a **conditional step** in Process.xaml

```
Process.xaml flow:
1. DownloadInvoice.xaml
2. ExtractInvoiceData.xaml
3. ValidateAgainstPO.xaml
   ↓
   IF validation fails (out_IsValid = False):
     4a. AnalyzeExceptionWithAI.xaml ← NEW
         ↓
         IF recommendation = "Auto-Resolve":
           5a. Apply correction (update vendor name or apply tolerance)
           6a. UploadToQuickBooks.xaml
         ELSE:
           5b. Add to InvoiceExceptions queue with AI reasoning
   ELSE:
     4b. CheckApprovalThreshold.xaml
     5b. UploadToQuickBooks.xaml
```

Does this integration approach work for you?

### Engineer
Yes, that's perfect. I like that the AI agent sits cleanly between validation failure and exception routing. If AI can fix it, we continue the happy path. If not, we route to manual review with context.

### Spec Agent
Excellent! Now let me outline the implementation plan.

**I'll create two deliverables for you:**

1. **Plan.md** - Autopilot-ready prompts for building the RPA workflow
2. **TestScenarios.md** - Test scenarios and expected outcomes

**The Plan.md will include:**
- Autopilot prompt for `AnalyzeExceptionWithAI.xaml` workflow
- Autopilot prompt for updating `Process.xaml` (integration point)
- **Agent Builder configuration** for the `InvoiceExceptionAnalyzer` AI agent

**Key Design Decisions:**

### Decision 1: AI Agent Response Format
The AI agent will return structured JSON:
```json
{
  "recommended_action": "Auto-Resolve",
  "reasoning": "Vendor 'Acme Inc' is a known alias...",
  "confidence": 0.92,
  "exception_type": "VendorAlias",
  "correction_data": {
    "corrected_vendor_name": "Acme Incorporated"
  }
}
```

This allows the RPA workflow to programmatically apply corrections.

### Decision 2: Graceful Degradation Strategy
```
TRY:
  Invoke AI Agent (5s timeout)
  Parse response
  Apply recommendation
CATCH (Timeout, HTTP error, JSON parsing error):
  Log warning: "AI agent unavailable, routing to manual review"
  Add to InvoiceExceptions queue (current behavior)
FINALLY:
  Continue processing (never throw exception)
```

### Decision 3: Configuration-Driven Thresholds
All magic numbers from Config.xlsx:
- `AIAutoResolveThreshold` = 0.85
- `AIMaxAutoResolveAmount` = 5000
- `AIAmountTolerancePercent` = 5

This allows tuning without redeployment.

Do these design decisions align with your requirements?

### Engineer
Perfect! I especially like the graceful degradation in the Try-Catch. That ensures we never break the automation.

One question: For the "correction_data" - will the AI agent know what the corrected vendor name should be?

### Spec Agent
Great question! Yes, the AI agent will have access to:

1. **Known Vendor Alias Database** (top 20 vendors seed data)
2. **Historical Exception Resolutions** (12 months of data showing what AP clerks changed vendor names to)
3. **SAP PO Data** (the correct vendor name from the PO)

So when the AI detects a vendor alias issue, it can return:
```json
"correction_data": {
  "corrected_vendor_name": "Acme Incorporated",  ← from PO or known aliases
  "source": "SAP_PO"  ← indicates where it found the correct name
}
```

For amount mismatches, it would return:
```json
"correction_data": {
  "tolerance_applied": 5.0,  ← percentage difference
  "adjusted_amount": 1050.00  ← actual invoice amount (within tolerance)
}
```

This tells the RPA workflow exactly what to update.

### Engineer
That's perfect! The workflow can just apply the correction without additional logic.

### Spec Agent
Exactly! Now let me ask about the AI agent itself.

**AI Agent Builder Configuration:**

The InvoiceExceptionAnalyzer agent needs:

1. **System Prompt** - Defines the agent's role and decision-making framework
2. **Tools** - Functions the agent can call (SAP PO lookup, QuickBooks duplicate check, alias database)
3. **Knowledge Base** - Historical exception data (12 months)
4. **Guardrails** - Business rules (no auto-approve >$5K, etc.)

**Should I include:**
- Full Agent Builder prompt in the Plan.md?
- Step-by-step instructions for deploying the agent?
- Or just high-level configuration and you'll handle Agent Builder setup separately?

### Engineer
Include the full Agent Builder prompt and configuration in Plan.md. I want everything in one place. I can follow the instructions to set up the agent in Agent Builder while I'm building the workflow.

### Spec Agent
Perfect! I'll include a complete Agent Builder section with:
- System prompt template
- Tool definitions (with sample code)
- Knowledge base structure
- Guardrail configuration

**One more question about testing:**

For TestScenarios.md, should I include:
- Unit test scenarios (just the AnalyzeExceptionWithAI.xaml workflow)?
- Integration test scenarios (AI agent + workflow together)?
- End-to-end test scenarios (full invoice processing with AI)?
- All of the above?

### Engineer
All of the above. I need comprehensive test coverage for:
1. **Unit tests** - Workflow logic with mocked AI responses
2. **Integration tests** - Real AI agent integration with known test cases
3. **E2E tests** - Full invoice flow from download to resolution

Also, include the **shadow mode validation** scenario - where AI logs recommendations but doesn't auto-resolve yet.

### Spec Agent
Excellent - comprehensive test coverage. I'll organize TestScenarios.md by:
- Unit Tests (mocked AI)
- Integration Tests (real AI, specific scenarios)
- E2E Tests (full flow)
- Shadow Mode Validation (observation mode)

**Final clarification:**

For the Autopilot prompts, how detailed should they be?

**Option A:** High-level (conceptual)
> "Create a workflow that invokes an AI agent and handles the response..."

**Option B:** Detailed (specific activities and logic)
> "Create a workflow with these activities: 1) Build DataTable for AI context, 2) Serialize to JSON, 3) Invoke AI Agent with timeout, 4) Deserialize response, 5) Apply business rules..."

Which level of detail works best for Autopilot?

### Engineer
Option B - detailed and specific. Autopilot works much better with concrete activity names and step-by-step logic. Include:
- Exact activity names (modern design activities)
- Argument names and types
- Variable names and scopes
- Error handling Try-Catch blocks
- Log messages

The more specific, the better the Autopilot-generated workflow will be.

### Spec Agent
Perfect! I'll create highly detailed, prescriptive prompts.

**Summary of what I'll deliver:**

### Plan.md Contents:
1. **Architecture Overview** - Integration point diagram
2. **Workflow Specification: AnalyzeExceptionWithAI.xaml**
   - Detailed Autopilot prompt with all activities
   - Arguments (in/out)
   - Variables
   - Logic flow with Try-Catch
3. **Workflow Modification: Process.xaml**
   - Autopilot prompt for adding conditional AI invocation
4. **AI Agent Builder Configuration**
   - Complete system prompt
   - Tool definitions (SAP lookup, QB duplicate check, alias DB)
   - Knowledge base structure
   - Guardrail rules
5. **Configuration Updates**
   - New settings for Config.xlsx
   - New Orchestrator assets
6. **Deployment Steps**
   - Agent Builder deployment
   - Workflow deployment
   - Shadow mode → Live mode transition

### TestScenarios.md Contents:
1. **Unit Tests** (5-6 scenarios)
2. **Integration Tests** (8-10 scenarios covering all exception types)
3. **E2E Tests** (4-5 scenarios)
4. **Shadow Mode Validation** (observation mode test plan)
5. **Expected Results** for each scenario

Ready to generate! Any final adjustments?

### Engineer
Looks perfect! Go ahead and generate both documents.

### Spec Agent
Generating Plan.md and TestScenarios.md now...

---

**[Spec Agent generates Plan.md and TestScenarios.md - see next files]**
