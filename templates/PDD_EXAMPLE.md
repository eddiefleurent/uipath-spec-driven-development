# Process Definition Document: Invoice Approval Process

**Version:** 1.0
**Process Owner:** Accounts Payable Department
**Created By:** Business Analyst
**Created Date:** January 2026
**Documentation Method:** UiPath Task Capture (or other process documentation method)

---

## 1. Introduction

### 1.1 Purpose

This document describes the AS-IS invoice approval process currently performed manually by the Accounts Payable team. It serves as the foundation for automation analysis and requirements gathering.

### 1.2 Objectives

| Objective | Description |
|-----------|-------------|
| **Primary** | Document current manual process for automation assessment |
| **Secondary** | Identify pain points, exceptions, and automation opportunities |
| **Tertiary** | Establish baseline metrics for ROI calculation |

### 1.3 Process Contacts

| Role | Name | Department | Responsibility |
|------|------|------------|----------------|
| Process Owner | Jane Smith | Accounts Payable | Final approval authority |
| Subject Matter Expert | John Doe | AP Operations | Day-to-day execution |
| IT Contact | Mike Johnson | IT Operations | System access and support |
| Business Analyst | Sarah Williams | CoE | Documentation and analysis |

### 1.4 Prerequisites for Automation

| Prerequisite | Status | Notes |
|--------------|--------|-------|
| Stable process | ✓ | Process unchanged for 18 months |
| Digital inputs | ✓ | Invoices received as PDF via email |
| Rule-based decisions | ✓ | Clear approval thresholds |
| System access available | ✓ | API access to SAP and QuickBooks |
| High volume | ✓ | 50-100 invoices/day |

---

## 2. Process Documentation

### 2.1 AS-IS Process Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Invoice Approval Process                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌───────────────┐  │
│  │ Receive  │───▶│ Download &   │───▶│ Extract     │───▶│ Validate      │  │
│  │ Email    │    │ Save Invoice │    │ Invoice     │    │ Against PO    │  │
│  └──────────┘    └──────────────┘    │ Details     │    └───────┬───────┘  │
│                                       └─────────────┘            │          │
│                                                                  ▼          │
│                                                         ┌───────────────┐   │
│                                                         │ PO Match?     │   │
│                                                         └───────┬───────┘   │
│                                              ┌──────────────────┴───────┐   │
│                                              │                          │   │
│                                              ▼                          ▼   │
│                                     ┌─────────────┐            ┌──────────┐ │
│                                     │ Route for   │            │ Check    │ │
│                                     │ Manual      │            │ Approval │ │
│                                     │ Review      │            │ Threshold│ │
│                                     └─────────────┘            └────┬─────┘ │
│                                                                     │       │
│                                              ┌──────────────────────┴────┐  │
│                                              │                           │  │
│                                              ▼                           ▼  │
│                                     ┌─────────────┐           ┌───────────┐ │
│                                     │ Manager     │           │ Auto      │ │
│                                     │ Approval    │           │ Approve   │ │
│                                     └──────┬──────┘           └─────┬─────┘ │
│                                            │                        │       │
│                                            └────────────┬───────────┘       │
│                                                         ▼                   │
│                                                ┌─────────────────┐          │
│                                                │ Upload to       │          │
│                                                │ QuickBooks      │          │
│                                                └─────────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Process Statistics

| Metric | Value | Notes |
|--------|-------|-------|
| **Daily Volume** | 50-100 invoices | Higher at month-end |
| **Average Processing Time** | 8-12 minutes/invoice | Manual process |
| **Error Rate** | 5-8% | Data entry errors |
| **FTE Involved** | 3 | Full-time AP clerks |
| **Peak Hours** | 9 AM - 11 AM | When emails arrive |
| **SLA** | Same business day | Must process by 5 PM |

### 2.3 Applications Used

| Application | Purpose | Access Method |
|-------------|---------|---------------|
| Microsoft Outlook | Receive invoice emails | Desktop application |
| Acme Vendor Portal | Download invoices | Web browser (Chrome) |
| SAP S/4HANA | Validate against PO | Web API |
| QuickBooks Online | Upload approved invoices | Web API |
| SharePoint | Store processed invoices | File system |
| Excel | Track processing status | Desktop application |

---

## 3. Detailed Process Steps

### Step 1: Receive Invoice Email

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | Microsoft Outlook |
| **Action Type** | Manual |
| **Timeframe** | 30 seconds |

**Description:**
AP Clerk monitors the shared AP inbox (ap@company.com) for new invoice emails from vendors. Emails typically have subject line "Invoice #[number] from [Vendor Name]" and contain PDF attachment.

**Screenshot Reference:** [Outlook inbox showing invoice emails]

**Business Rules:**
- Only process emails from known vendor domains
- Ignore emails without PDF attachments
- Flag suspicious emails for security review

---

### Step 2: Download Invoice from Vendor Portal

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | Acme Vendor Portal (Chrome) |
| **Action Type** | Manual - UI Navigation |
| **Timeframe** | 2-3 minutes |

**Description:**
1. Open Chrome browser
2. Navigate to https://portal.acmevendors.com
3. Log in with service account credentials
4. Navigate to "Invoices" → "New/Unpaid"
5. Click on invoice row to open details
6. Click "Download PDF" button
7. Save to network drive: `\\server\AP\Invoices\YYYY-MM\`

**Screenshot Reference:** [Vendor portal invoice list and download button]

**Business Rules:**
- Use naming convention: `VendorName_InvoiceNumber_YYYYMMDD.pdf`
- Only download invoices with status "New" or "Unpaid"
- Skip invoices already downloaded (check by invoice number)

**Decision Point:**
- If invoice already exists in folder → Skip download
- If portal unavailable → Retry in 15 minutes, then escalate to IT

---

### Step 3: Extract Invoice Details

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | PDF Reader / Excel |
| **Action Type** | Manual - Data Entry |
| **Timeframe** | 3-4 minutes |

**Description:**
1. Open downloaded PDF invoice
2. Locate and read key fields:
   - Invoice Number (top right, format: INV-XXXXX)
   - Invoice Date (below invoice number)
   - Vendor Name (header section)
   - PO Number (reference section, format: PO-XXXXX)
   - Line items (table in body)
   - Total Amount (bottom right, before tax)
   - Tax Amount (if applicable)
   - Grand Total (final amount due)
3. Enter data into tracking spreadsheet

**Screenshot Reference:** [Sample invoice with fields highlighted]

**Data Fields:**

| Field | Location | Format | Required |
|-------|----------|--------|----------|
| Invoice Number | Top right | INV-XXXXX | Yes |
| Invoice Date | Below invoice # | MM/DD/YYYY | Yes |
| Vendor Name | Header | Text | Yes |
| PO Number | Reference section | PO-XXXXX | Yes |
| Total Amount | Bottom right | $X,XXX.XX | Yes |
| Tax Amount | Below total | $X,XXX.XX | No |
| Grand Total | Final line | $X,XXX.XX | Yes |

**Business Rules:**
- All required fields must be present
- Invoice date cannot be future-dated
- Amount must be positive
- PO number must match expected format

---

### Step 4: Validate Against Purchase Order

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | SAP S/4HANA |
| **Action Type** | Manual - System Lookup |
| **Timeframe** | 2-3 minutes |

**Description:**
1. Log into SAP S/4HANA
2. Navigate to Purchase Order module
3. Search for PO number from invoice
4. Compare invoice details against PO:
   - Vendor name must match
   - Line items must match
   - Total amount must match (within 2% tolerance)
   - PO must be in "Open" status
5. Document validation result

**Screenshot Reference:** [SAP PO lookup screen]

**Validation Rules:**

| Check | Rule | Action if Failed |
|-------|------|------------------|
| Vendor Match | Invoice vendor = PO vendor | Route to manual review |
| Amount Match | Invoice total within 2% of PO total | Route to manual review |
| PO Status | PO must be "Open" | Route to manual review |
| PO Exists | PO number found in SAP | Route to manual review |

**Decision Point:**
- If all validations pass → Proceed to Step 5
- If any validation fails → Route to Step 6 (Manual Review)

---

### Step 5: Check Approval Threshold

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk / System |
| **Application** | Business Rules |
| **Action Type** | Decision |
| **Timeframe** | 30 seconds |

**Description:**
Determine if invoice requires manager approval based on amount thresholds.

**Approval Matrix:**

| Amount Range | Approval Required | Approver |
|--------------|-------------------|----------|
| $0 - $1,000 | None (Auto-approve) | System |
| $1,001 - $10,000 | Single approval | AP Manager |
| $10,001 - $50,000 | Single approval | Finance Director |
| $50,001+ | Dual approval | CFO + Finance Director |

**Decision Point:**
- If amount ≤ $1,000 → Auto-approve, proceed to Step 7
- If amount > $1,000 → Route to Step 6 (Manager Approval)

---

### Step 6: Manager Approval (If Required)

| Property | Value |
|----------|-------|
| **Actor** | AP Manager / Finance Director / CFO |
| **Application** | Email / Approval System |
| **Action Type** | Manual - Review & Approve |
| **Timeframe** | 4-24 hours (SLA dependent) |

**Description:**
1. AP Clerk sends approval request email with:
   - Invoice PDF attached
   - Summary of invoice details
   - PO validation results
   - Recommended action
2. Approver reviews and responds:
   - "Approved" → Proceed to Step 7
   - "Rejected" → Return to vendor with reason
   - "More Info" → AP Clerk provides additional details

**Screenshot Reference:** [Approval email template]

**Business Rules:**
- Approval requests must include all supporting documentation
- Approvers have 24-hour SLA for amounts under $10,000
- Approvers have 48-hour SLA for amounts over $10,000
- Escalate if no response within SLA

**Decision Point:**
- If approved → Proceed to Step 7
- If rejected → Notify vendor, close process
- If more info requested → Gather info, resubmit

---

### Step 7: Upload to QuickBooks

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | QuickBooks Online |
| **Action Type** | Manual - Data Entry |
| **Timeframe** | 2-3 minutes |

**Description:**
1. Log into QuickBooks Online
2. Navigate to Expenses → Bills
3. Click "Create Bill"
4. Enter invoice details:
   - Vendor (select from dropdown)
   - Bill Date (invoice date)
   - Due Date (calculated from payment terms)
   - Bill Number (invoice number)
   - Line items (from invoice)
   - Total amount
5. Attach PDF invoice
6. Click "Save and Close"

**Screenshot Reference:** [QuickBooks bill entry screen]

**Business Rules:**
- Due date = Invoice date + Payment terms (Net 30 default)
- Must attach original PDF for audit trail
- Verify total matches before saving

---

### Step 8: Update Tracking & Archive

| Property | Value |
|----------|-------|
| **Actor** | AP Clerk |
| **Application** | Excel / SharePoint |
| **Action Type** | Manual - Documentation |
| **Timeframe** | 1-2 minutes |

**Description:**
1. Update tracking spreadsheet with:
   - Processing date/time
   - Status (Completed/Rejected/Pending)
   - QuickBooks reference number
   - Any notes or exceptions
2. Move PDF to "Processed" folder on SharePoint
3. Mark email as complete in Outlook

**Screenshot Reference:** [Tracking spreadsheet]

---

## 4. To-Be Process Analysis

### 4.1 To-Be Process Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Invoice Approval Process (Automated)                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌───────────────┐  │
│  │ Monitor  │───▶│ Download &   │───▶│ Extract via │───▶│ Validate via  │  │
│  │ Inbox    │    │ Save (Auto)  │    │ Document    │    │ SAP API       │  │
│  │ (Auto)   │    └──────────────┘    │ Understanding│   └───────┬───────┘  │
│  └──────────┘                        └─────────────┘            │          │
│                                                                  ▼          │
│                                                         ┌───────────────┐   │
│                                                         │ Auto-Approve  │   │
│                                                         │ or Route      │   │
│                                                         └───────┬───────┘   │
│                                              ┌──────────────────┴───────┐   │
│                                              │                          │   │
│                                              ▼                          ▼   │
│                                     ┌─────────────┐           ┌───────────┐ │
│                                     │ Human       │           │ Upload to │ │
│                                     │ Review      │           │ QuickBooks│ │
│                                     │ (Exception) │           │ (Auto)    │ │
│                                     └─────────────┘           └───────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Automation Opportunities

| Step | Current | Proposed | Benefit |
|------|---------|----------|---------|
| Email monitoring | Manual check | Automated trigger | Real-time processing |
| Invoice download | Manual navigation | Automated via portal | Time savings |
| Data extraction | Manual reading | Document Understanding | Accuracy, speed |
| PO validation | Manual lookup | SAP API call | Instant validation |
| Threshold check | Manual decision | Business rules engine | Consistency |
| QuickBooks upload | Manual entry | API integration | Error reduction |

### 4.3 Parallel Initiatives

| Initiative | Status | Impact on Automation |
|------------|--------|---------------------|
| SAP S/4HANA upgrade | Planned Q2 | May change API endpoints |
| QuickBooks migration | None | No impact |
| Vendor portal redesign | In progress | May affect selectors |

### 4.4 In Scope for RPA

| Item | In Scope | Notes |
|------|----------|-------|
| Email monitoring | ✓ | Trigger-based |
| Invoice download | ✓ | UI automation |
| Data extraction | ✓ | Document Understanding |
| PO validation | ✓ | API integration |
| Auto-approval (≤$1,000) | ✓ | Business rules |
| QuickBooks upload | ✓ | API integration |
| Tracking update | ✓ | Database/file update |

### 4.5 Out of Scope for RPA

| Item | Reason |
|------|--------|
| Manager approval decisions | Requires human judgment |
| Vendor dispute resolution | Complex negotiation |
| New vendor onboarding | One-time setup |
| Payment execution | Separate process |

---

## 5. Exception Planning

### 5.1 Business Exception Handling

| Exception | Frequency | Current Handling | Proposed Handling |
|-----------|-----------|------------------|-------------------|
| Missing PO number | 5% | Manual lookup | Route to exception queue |
| Amount mismatch | 3% | Manager review | Route to exception queue |
| Unknown vendor | 1% | Reject & contact | Route to exception queue |
| Duplicate invoice | 2% | Skip processing | Auto-detect & skip |
| Invalid date | <1% | Correct manually | Route to exception queue |

### 5.2 Application Error Procedures

| Error | Frequency | Current Handling | Proposed Handling |
|-------|-----------|------------------|-------------------|
| Portal unavailable | Weekly | Wait & retry | Auto-retry 3x, then alert |
| SAP timeout | Rare | Refresh & retry | Auto-retry with backoff |
| QuickBooks API error | Rare | Manual entry | Retry once, then queue |
| PDF corrupted | <1% | Request new copy | Log & alert, skip |

### 5.3 Reporting Requirements

| Report | Frequency | Audience | Content |
|--------|-----------|----------|---------|
| Daily summary | Daily | AP Team | Processed count, exceptions |
| Exception report | Daily | AP Manager | Failed items, reasons |
| Weekly metrics | Weekly | Finance Director | Volume, SLA compliance |
| Monthly audit | Monthly | Compliance | Full processing log |

---

## 6. Appendix

### 6.1 Glossary

| Term | Definition |
|------|------------|
| PO | Purchase Order - authorization to purchase |
| AP | Accounts Payable - department handling vendor payments |
| SLA | Service Level Agreement - processing time commitment |
| FTE | Full-Time Equivalent - staffing measure |

### 6.2 Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Jan 2026 | Sarah Williams | Initial creation via Task Capture |

### 6.3 Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Process Owner | Jane Smith | | |
| SME | John Doe | | |
| Business Analyst | Sarah Williams | | |

---

*This document was created using UiPath Task Capture and follows the standard PDD template format.*
