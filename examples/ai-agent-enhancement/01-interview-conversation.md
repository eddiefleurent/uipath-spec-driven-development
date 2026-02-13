# Interview Agent Conversation

**Session Date:** February 1, 2026
**Engineer:** Mike Chen
**Agent:** Interview Agent (BA Expert persona)

---

## Conversation Transcript

### Engineer
Hi, I have a new user story to work on: IAUTO-247 - Add AI-powered exception analysis to our Invoice Approval automation. Can you help me gather requirements?

Here's the user story: [provides 00-user-story.md content]

### Interview Agent
Thank you for providing the user story! I've reviewed it and I have access to your existing documentation:

**Context Documents I've Read:**
- ✓ PDD.md - Invoice Approval Process (AS-IS business process)
- ✓ TDD.md v1.0 - Invoice Approval Automation (current architecture without AI agent)
- ✓ User Story IAUTO-247

**Understanding So Far:**
You want to enhance the existing Invoice Approval automation (currently v1.0, REFramework-based) by adding an AI agent that analyzes validation exceptions and auto-resolves common issues. The goal is to reduce manual review from 30% to 20% of invoices.

**I have a few clarifying questions to ensure we capture complete requirements:**

### Question 1: AI Agent Invocation Point
Looking at your current TDD.md, I see the validation happens in `ValidateAgainstPO.xaml`. When exactly should the AI agent be invoked?

A) After **any** validation failure in ValidateAgainstPO.xaml
B) Only for **specific** validation failure types (vendor mismatch, amount mismatch, etc.)
C) As a **final step** before routing to manual review queue

What's your preference?

### Engineer
Option C makes the most sense - invoke the AI agent as a final step before routing to manual review. This way it acts as an intelligent filter that tries to resolve exceptions before they hit the human queue.

### Interview Agent
Perfect, that's a clean integration point. The AI agent will sit between validation failure and the exception queue.

### Question 2: Auto-Resolution Scope
The user story mentions several types of auto-resolvable exceptions. Let me confirm the priority and confidence thresholds:

**For Phase 1 (MVP), which exception types should the AI handle?**

Looking at your PDD and exception patterns:
- Vendor name variations (e.g., "Acme Inc" vs "Acme Incorporated")
- Amount mismatches within 5% tolerance
- Duplicate invoice detection
- PO not found (investigate if alias issue)

Should we implement **all** of these in Phase 1, or start with a subset?

### Engineer
Let's start with all of them for Phase 1. Jane wants to hit the 30% reduction target, and based on the exception report, we need all four types to get there.

### Interview Agent
Got it - all four exception types in scope for Phase 1.

### Question 3: Knowledge Base & Historical Data
To make the AI agent effective, it needs training data. I see from your TDD that invoices go to QuickBooks after approval.

**What historical data should we use to train the agent?**

A) Last 12 months of manual exception resolutions (from exception queue tracking)
B) Last 6 months only (more recent patterns)
C) Custom dataset curated by AP team

Also, do you have structured data on vendor aliases, or does the AI need to learn these patterns?

### Engineer
We have 12 months of exception queue data in Excel - it includes the exception reason, what the AP clerk did to resolve it, and the outcome. Let's use all 12 months.

For vendor aliases, we have a partial list (top 20 vendors), but many aliases aren't documented. The AI should be able to learn patterns from the historical data.

### Interview Agent
Excellent - 12 months of historical resolutions plus the partial vendor alias list. The agent can augment the known aliases by learning from patterns.

### Question 4: Confidence Thresholds & Guardrails
The user story mentions >85% confidence for auto-resolution. Let me confirm the decision matrix:

**Based on confidence score, what should happen?**

Current understanding:
- Confidence > 85%: Auto-resolve, proceed to QuickBooks
- Confidence 75-85%: Route to manual review **with** AI reasoning
- Confidence < 75%: Route to manual review (standard flow)

Is this correct? Any other guardrails?

### Engineer
Yes, that's exactly right. One additional guardrail: the AI agent should **never** auto-approve invoices over $5,000, regardless of confidence. Jane was very clear about that - high-value invoices need human eyes.

### Interview Agent
Important guardrail noted - AI cannot auto-resolve if invoice amount > $5,000. This aligns with your existing approval matrix.

### Question 5: Fallback & Error Handling
You mentioned graceful degradation in the acceptance criteria. Let me confirm the fallback behavior:

**If the AI agent fails (timeout, unavailable, error), what should happen?**

A) Route directly to manual review queue (current behavior)
B) Retry the AI agent call 2-3 times, then fallback
C) Queue for later AI analysis in batch mode

What's the preferred approach?

### Engineer
Option A - just route directly to manual review. We can't afford delays waiting for AI retries. The manual review process already works, so fallback should be immediate. We should log a warning so we know the AI is having issues, but don't block the invoice.

### Interview Agent
Perfect - immediate fallback to manual review with warning logging. This maintains your current SLA.

### Question 6: Integration with Existing Workflows
Looking at your TDD.md architecture, I see you're using REFramework with several atomic workflows:
- DownloadInvoice.xaml
- ExtractInvoiceData.xaml
- ValidateAgainstPO.xaml
- CheckApprovalThreshold.xaml
- UploadToQuickBooks.xaml

**For the AI agent integration, should we:**

A) Create a new atomic workflow `AnalyzeExceptionWithAI.xaml` that's invoked from Process.xaml
B) Modify the existing ValidateAgainstPO.xaml to include AI logic
C) Add AI logic directly in the Process.xaml Try-Catch

Which approach fits your architecture standards?

### Engineer
Option A - create a new atomic workflow called `AnalyzeExceptionWithAI.xaml`. We don't want to modify existing workflows that are already tested and working. Keep it modular.

### Interview Agent
Excellent choice - maintains separation of concerns and testability.

### Question 7: Testing & Validation
For acceptance testing, how should we validate that the AI is making good decisions?

**Should we:**
- Run in "shadow mode" first (AI makes recommendations but humans still review everything)?
- Compare AI decisions against a held-out test set of known exceptions?
- Pilot with specific vendors before full rollout?

What's the validation approach?

### Engineer
Good question. Let's do a two-phase rollout:

**Phase 1 (Week 1-2):** Shadow mode - AI analyzes exceptions and logs recommendations, but all exceptions still go to manual review. Sarah (AP clerk) will validate AI recommendations against her decisions.

**Phase 2 (Week 3+):** Live mode - AI auto-resolves when confidence > 85%, starting with top 5 vendors only. After 2 weeks of monitoring, expand to all vendors.

### Interview Agent
Smart rollout strategy - shadow mode for validation, then phased go-live.

### Question 8: Outputs & Reporting
What information should the AI agent return to support auditing and continuous improvement?

**Should we capture:**
- Recommended action (Auto-Resolve, Route-to-AP-Manager, Reject, etc.)
- Natural language reasoning (why this recommendation?)
- Confidence score
- Exception type classification
- Any other metadata?

### Engineer
All of the above. Jane wants to review AI decisions weekly, so we need:
- Recommended action
- Reasoning (natural language explanation)
- Confidence score
- Exception type (vendor alias, amount mismatch, duplicate, etc.)
- Invoice details (number, vendor, amount)
- Timestamp

This should all be logged and also attached to the exception queue item if it's routed to manual review.

### Interview Agent
Perfect - comprehensive logging for audit trail and continuous improvement.

### Final Question: Timeline & Dependencies
Looking at the user story, Jane wants results for Q2 business review (March 31).

**Do you have:**
- Access to UiPath Agent Builder? (for building the InvoiceExceptionAnalyzer agent)
- Access to UiPath AI Center? (for hosting the agent)
- Access to the 12 months of exception data?
- Approval to create test invoices for validation?

Any blockers I should document?

### Engineer
Yes to all:
- Agent Builder access: ✓ (license acquired last month)
- AI Center: ✓ (same license)
- Historical data: ✓ (I have the Excel file from Sarah)
- Test data: ✓ (we have a test vendor account in SAP)

No blockers. Timeline is tight but achievable if we scope to the four exception types we discussed.

---

## Interview Agent Summary

Thank you, Mike! I have everything I need. Let me summarize what I've captured:

**Enhancement Scope:**
- Add AI agent to existing Invoice Approval automation (v1.0 → v2.0)
- Invocation point: After validation failure, before manual review queue
- New atomic workflow: `AnalyzeExceptionWithAI.xaml`

**AI Agent Capabilities:**
1. Analyze four exception types: vendor alias, amount mismatch, duplicate, PO issues
2. Auto-resolve when confidence > 85% AND amount ≤ $5,000
3. Route to manual review with reasoning when confidence 75-85% or amount > $5,000
4. Fallback to manual review immediately if AI fails

**Data & Learning:**
- Historical: 12 months of exception queue resolutions
- Known aliases: Top 20 vendors (partial list)
- Pattern learning: Agent learns additional aliases from historical data

**Guardrails:**
- Cannot auto-resolve invoices > $5,000
- Must return recommendation within 5 seconds (timeout)
- Graceful degradation to manual review on failure

**Rollout:**
- Phase 1: Shadow mode (2 weeks) - validation only
- Phase 2: Live mode - top 5 vendors, then expand

**Logging & Audit:**
- Recommended action, reasoning, confidence, exception type, invoice details, timestamp

I'll now generate the Requirements.md document. Please review it and let me know if I missed anything!

### Engineer
Looks perfect! Go ahead and generate Requirements.md.

---

**[Interview Agent generates Requirements.md - see next file]**
