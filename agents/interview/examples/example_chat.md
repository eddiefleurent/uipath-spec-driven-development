# Example Interview: Invoice Processing Automation

This example demonstrates a complete interview session using the Interview Agent, showing how the agent uses PDD.md to focus the conversation.

---

## Initial User Story

> "We need to automate the process of downloading invoices from our vendor portal (Acme Vendor Portal), extracting key fields (invoice number, date, amount, vendor name), validating against our purchase order system (SAP), and if valid, uploading to our accounting system (QuickBooks). Invoices are in PDF format. This needs to run daily at 6 AM."

---

## Context Provided

- **PDD.md**: Process Definition Document for Invoice Approval Process (see [templates/PDD_EXAMPLE.md](../../../templates/PDD_EXAMPLE.md))
- **TDD.md**: Not provided (new automation project)

---

## Interview Session

### Phase 1: Initial Analysis (With PDD)

**Agent:**

Thank you for providing the user story and the PDD. I've reviewed the Process Definition Document for the Invoice Approval Process.

**From the PDD, I understand:**
- The current process handles 50-100 invoices daily
- Invoices are downloaded from Acme Vendor Portal (filtered by "New/Unpaid" status)
- Key fields extracted: Invoice Number, Invoice Date, Vendor Name, PO Number, Total Amount
- Validation against SAP requires: vendor match, amount within 2% tolerance, PO in "Open" status
- Approval thresholds: ≤$1,000 auto-approve, >$1,000 requires manager approval
- QuickBooks upload is done via API
- Exception handling includes retry logic for portal unavailability

**This user story aligns with Steps 1-4 and Step 7 of the documented process.**

Since the PDD provides comprehensive process details, I'll focus my questions on:
1. Gaps not covered in the PDD
2. Automation-specific requirements
3. Clarifications on edge cases

Let's proceed with targeted questions.

---

### Phase 2: Requirements Gathering (Focused Questions)

**Agent:**

Based on the PDD, I already have answers to many standard questions. Let me ask about **gaps and automation-specific details**:

#### 1. Automation-Specific Questions (Not in PDD)
- The PDD mentions service account credentials for the portal. Where should the automation store these credentials? (Orchestrator Assets, Azure Key Vault, etc.)
- The PDD shows a 30-minute SLA. Should the automation have a timeout, or should it process all invoices regardless of time?
- Should this run as attended or unattended automation?

#### 2. Clarifications on PDD Details
- The PDD mentions invoices are "semi-structured but consistent per vendor." Do all vendors use the same invoice template, or are there multiple templates to handle?
- The PDD shows 2% tolerance for amount matching. Is this tolerance configurable, or should it be hardcoded?

#### 3. Scope Confirmation
- The PDD includes manager approval for invoices >$1,000. Should this automation handle the approval workflow, or just the auto-approve path (≤$1,000)?
- Should the automation update the tracking spreadsheet mentioned in Step 8, or will a database replace this?

#### 4. Error Handling Confirmation
- The PDD specifies "retry 3x then alert" for portal issues. Who should receive these alerts? (Email distribution list, Teams channel, etc.)
- For SAP validation failures, the PDD says "flag and continue." Should these be added to an Orchestrator queue for manual review?

*Note: I'm skipping questions about process steps, business rules, and basic error handling since the PDD already documents these clearly.*

---

**User:**

**1. Automation-Specific:**
- Store credentials in Orchestrator Assets (Credential type)
- No hard timeout—process all invoices, but alert if exceeding 30 minutes
- Unattended automation, triggered by Orchestrator schedule

**2. Clarifications:**
- We have 3 main vendor templates, but they're similar enough that Document Understanding should handle them
- The 2% tolerance should be configurable via Config.xlsx

**3. Scope:**
- For now, only handle auto-approve path (≤$1,000). Manager approval will be Phase 2.
- Replace the tracking spreadsheet with a SQL database for better reporting

**4. Error Handling:**
- Alerts go to ap-alerts@company.com distribution list
- Yes, add validation failures to an Orchestrator queue called "InvoiceExceptions" for manual review

---

### Phase 3: Validation

**Agent:**

Thank you! Let me summarize the requirements, combining PDD context with your clarifications:

#### From PDD (Relevant Context for This Story)
- **Volume:** 50-100 invoices daily (PDD Step 2.2)
- **Process Steps:** Download → Extract → Validate → Upload (PDD Steps 2-4, 7)
- **Validation Rules:** Vendor match, amount within 2% tolerance, PO status "Open" (PDD Step 4)
- **Business Rules:** Auto-approve ≤$1,000 (PDD Step 5)
- **Error Handling:** Retry 3x for portal issues, skip corrupt PDFs (PDD Section 5)

#### From This Interview (Automation-Specific)
- **Scope:** Auto-approve path only (≤$1,000); manager approval is Phase 2
- **Credentials:** Orchestrator Assets (Credential type)
- **Execution:** Unattended, Orchestrator-triggered at 6 AM
- **Data Storage:** SQL database (replacing tracking spreadsheet)
- **Configuration:** 2% tolerance configurable via Config.xlsx
- **Alerts:** ap-alerts@company.com distribution list
- **Exceptions:** Add to "InvoiceExceptions" Orchestrator queue

#### Combined Requirements Summary

| Aspect | Requirement |
|--------|-------------|
| Trigger | Orchestrator schedule, daily 6 AM |
| Scope | Auto-approve invoices ≤$1,000 only |
| Portal Access | Orchestrator Credential asset |
| Invoice Templates | 3 vendor templates, Document Understanding |
| Validation | SAP API, 2% tolerance (configurable) |
| Output | QuickBooks API upload |
| Exceptions | Orchestrator queue for manual review |
| Alerts | Email to ap-alerts@company.com |
| Audit | SQL database + retained PDFs |

**Does this capture the requirements accurately?**

---

**User:**

> Yes

---

### Phase 4: Document Generation

**Agent:**

Great! I've generated the complete Requirements.md document.

**[See [example_output.md](./example_output.md) for the full generated document]**

---

## Key Observations

This interview demonstrates several best practices:

| Practice | Example |
|----------|---------|
| **PDD-informed questions** | Agent skipped basic process questions already answered by PDD |
| **Focused on gaps** | Questions targeted automation-specific details not in PDD |
| **Selective extraction** | Agent identified relevant PDD sections (Steps 2-4, 7) for this story |
| **Explicit validation** | Summary clearly separates PDD context from interview findings |
| **Scope clarification** | Agent confirmed what's in/out of scope (manager approval = Phase 2) |

---

## Interview Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Question groups | 4 | Reduced from 5 (PDD covered business context) |
| Total questions asked | 12 | Reduced from 23 (PDD answered many) |
| User responses | 1 (grouped answers) | More efficient with PDD |
| Estimated duration | 3-5 minutes | Faster than without PDD |
| PDD sections referenced | 5 | Steps 2-4, 7, Section 5 |

---

## Comparison: With vs Without PDD

| Aspect | Without PDD | With PDD |
|--------|-------------|----------|
| Questions asked | 23 | 12 |
| Duration | 5-10 minutes | 3-5 minutes |
| Process understanding | Built from scratch | Pre-established |
| Focus | Basic process discovery | Automation-specific details |
| Requirements.md | Generic structure | Includes relevant PDD context |
