# Example Interview: Invoice Processing Automation

This example demonstrates a complete interview session using the Interview Agent.

---

## Initial User Story

> "We need to automate the process of downloading invoices from our vendor portal (Acme Vendor Portal), extracting key fields (invoice number, date, amount, vendor name), validating against our purchase order system (SAP), and if valid, uploading to our accounting system (QuickBooks). Invoices are in PDF format. This needs to run daily at 6 AM."

---

## Interview Session

### Phase 1: Initial Analysis

**Agent:**

Thank you for providing the details of the process you want to automate. I'll start by asking some clarifying questions to ensure I fully understand your requirements and can document them comprehensively.

From your description, the process involves:
1. Logging into the Acme Vendor Portal
2. Downloading invoices in PDF format
3. Extracting key fields from the invoices
4. Validating the extracted data against your purchase order system (SAP)
5. Uploading validated invoices to your accounting system (QuickBooks)
6. Scheduling the process to run daily at 6 AM

Let's move to the next phase to gather more details.

---

### Phase 2: Requirements Gathering

**Agent:**

#### 1. Business Context
- What is the primary goal of this automation? Is it to save time, reduce errors, or something else?
- How many invoices are typically processed daily?
- Are there any compliance or audit requirements for this process?

#### 2. Process Flow
- How do users currently log into the Acme Vendor Portal? Is it via username/password, SSO, or another method?
- Are there specific filters or criteria to identify which invoices need to be downloaded from the portal?
- Are the invoices stored in a specific folder or section within the portal?
- Should the downloaded invoices be saved locally or on a shared drive? If so, what is the folder structure or naming convention?
- What are the key fields to extract from the invoices? You mentioned invoice number, date, amount, and vendor name—are there any additional fields?
- How is the validation against SAP performed? Is there an API or database query involved, or is it a manual process currently?
- What specific criteria determine whether an invoice is valid or invalid?
- If an invoice is invalid, what should happen? Should it be flagged, logged, or sent to a specific team for review?
- How is the upload to QuickBooks performed? Is there an API, or does it involve manual data entry into the system?
- Are there any specific steps or approvals required before uploading to QuickBooks?

#### 3. Data & Integration
- Do you have access to the Acme Vendor Portal, SAP, and QuickBooks systems? Are there any access restrictions or permissions we need to consider?
- Are there existing APIs or connectors for SAP and QuickBooks that we can leverage for this automation?
- Are the PDF invoices structured (e.g., consistent format) or unstructured? If unstructured, do they vary significantly in layout?
- Are there any specific data validation rules for the extracted fields (e.g., date format, amount range)?
- Should the extracted data be stored in a database or file for reporting or audit purposes?

#### 4. Error Handling & Edge Cases
- What should happen if the Acme Vendor Portal is unavailable or the login fails?
- What should the automation do if an invoice cannot be downloaded or is corrupted?
- How should the process handle duplicate invoices?
- What should happen if the validation against SAP fails (e.g., no matching purchase order)?
- What should the automation do if the upload to QuickBooks fails?

#### 5. Technical Requirements
- Are there any specific security requirements for handling sensitive data (e.g., encryption, secure storage)?
- Should the automation send notifications (e.g., email alerts) for errors or completion? If so, who should receive them?
- Are there any performance requirements (e.g., maximum time to process all invoices)?
- Should the automation generate logs or reports? If so, what details should be included?

---

**User:**

**1. Business Context:**
- Primary goal is to reduce manual effort and errors
- We process ~20–40 invoices per day
- Basic audit trail is required (logs + retained PDFs), nothing extreme

**2. Process Flow:**
- Login is via username/password (no SSO, MFA not required for this account)
- Invoices are filtered by "New / Unpaid" status in the portal
- They're all in a single "Invoices" section, no subfolders
- Save PDFs to a shared network drive with naming: `Vendor_InvoiceNumber_Date.pdf`
- Fields needed are exactly: invoice number, invoice date, total amount, vendor name
- Validation against SAP is via API: invoice must match an open PO by PO number and amount (exact match)
- If invalid, mark as failed, log reason, and notify AP team by email
- QuickBooks upload is via API; no manual steps or approvals needed before upload

**3. Data & Integration:**
- We have service account access to Acme Portal, SAP APIs, and QuickBooks APIs
- PDFs are semi-structured but consistent per vendor (same template)
- Validation rules are straightforward: valid date, non-negative amount, required fields present
- Store extracted data + status in a database for reporting and audit

**4. Error Handling:**
- If portal login fails or portal is down: retry 3 times, then alert
- Corrupt or unreadable PDFs should be skipped, logged, and alerted
- Detect duplicates by invoice number + vendor; skip if already processed
- SAP validation failures should not stop the run—just flag and continue
- QuickBooks upload failures should retry once, then flag and alert

**5. Technical Requirements:**
- Credentials must be stored securely (vault/secret manager)
- Email notifications for failures and a daily success summary to AP distribution list
- End-to-end run should finish within 30 minutes
- Logs should include timestamp, invoice ID, step, status, and error message if any

---

### Phase 3: Validation

**Agent:**

Thank you for the detailed responses! Based on the information provided, I now have a clear understanding of the process and its requirements. Let me summarize:

#### Business Context
- **Goal:** Automate the invoice processing workflow to reduce manual effort and errors
- **Volume:** Approximately 20–40 invoices processed daily
- **Audit:** Basic audit trail required, including logs and retained PDFs

#### Process Flow
1. Log into Acme Vendor Portal using username and password
2. Filter invoices by "New / Unpaid" status and download from the "Invoices" section
3. Save PDFs to shared network drive with naming: `Vendor_InvoiceNumber_Date.pdf`
4. Extract key fields: invoice number, invoice date, total amount, vendor name
5. Validate via SAP API—match open PO by PO number and exact amount
6. If invalid: flag, log reason, notify AP team via email
7. If valid: upload to QuickBooks via API (no approval needed)

#### Data & Integration
- Service account access available for all systems
- PDFs are semi-structured but consistent per vendor
- Store extracted data and processing status in database for reporting/audit

#### Error Handling
| Scenario | Action |
|----------|--------|
| Portal login fails | Retry 3×, then alert |
| Corrupt/unreadable PDF | Skip, log, alert |
| Duplicate invoice | Skip if already processed |
| SAP validation failure | Flag and continue |
| QuickBooks upload failure | Retry once, then flag and alert |

#### Technical Requirements
- Credentials stored securely in vault/secret manager
- Email notifications for failures + daily success summary
- Complete within 30 minutes
- Comprehensive logging (timestamp, invoice ID, step, status, error message)

**Does this summary align with your expectations? If so, I'll proceed to generate the Requirements document.**

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
| **Grouped questions** | Questions organized by theme (Business, Process, Data, Errors, Technical) |
| **Follow-up based on answers** | Agent adapted questions based on API availability |
| **Explicit validation** | Summary presented before document generation |
| **Comprehensive coverage** | All critical areas addressed in ~5-10 minutes |

---

## Interview Metrics

| Metric | Value |
|--------|-------|
| Question groups | 5 |
| Total questions asked | 23 |
| User responses | 2 (grouped answers) |
| Estimated duration | 5-10 minutes |
| Completeness score | 95% |
