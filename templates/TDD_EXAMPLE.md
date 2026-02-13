# Technical Design Document: Invoice Approval Automation

**Version:** 2.0
**Status:** Production

---

## 1. Project Overview

### 1.1 Business Process

| Property | Value |
|----------|-------|
| **Process Name** | Invoice Approval Automation |
| **Business Owner** | Jane Smith, Accounts Payable Department |
| **Description** | Automates the invoice approval process from email receipt through QuickBooks upload, including PO validation, AI-powered exception analysis, and approval routing |
| **Trigger** | Orchestrator Queue (triggered by email arrival) |
| **Frequency** | Continuous (50-100 invoices/day) |

### 1.2 Scope

**In Scope:**
- Monitor shared AP inbox for invoice emails
- Download invoices from vendor portal
- Extract invoice data using Document Understanding
- Validate invoices against SAP purchase orders via API
- **Use AI Agent to analyze validation exceptions and recommend routing**
- Auto-approve invoices under $1,000
- Upload approved invoices to QuickBooks via API
- Update tracking spreadsheet and archive processed invoices

**Out of Scope:**
- Manager approval decisions (human task)
- Vendor dispute resolution
- New vendor onboarding
- Payment execution

---

## 2. Architecture

### 2.1 Pattern

| Property | Value |
|----------|-------|
| **Pattern** | REFramework (Queue-based) |
| **Rationale** | High transaction volume (50-100/day), each invoice is independent, requires retry logic for API timeouts, exception handling for validation failures |

### 2.2 Component Diagram

```
[Main.xaml - REFramework]
├── [Framework/InitAllApplications.xaml]
├── [Framework/GetTransactionData.xaml]
├── [Framework/Process.xaml]
│   ├── [DownloadInvoice.xaml]
│   ├── [ExtractInvoiceData.xaml]
│   ├── [ValidateAgainstPO.xaml]
│   ├── [AnalyzeExceptionWithAI.xaml] ◄── AI Agent Component
│   ├── [CheckApprovalThreshold.xaml]
│   └── [UploadToQuickBooks.xaml]
└── [Framework/CloseAllApplications.xaml]
```

### 2.3 Data Flow

```
Email Inbox → Queue Item → Download PDF → Document Understanding → SAP API Validation
                                                                           ↓
                                                                    [Validation Failed?]
                                                                           ↓
                                                              AI Agent Analysis & Routing
                                                                           ↓
                                                            [Auto-resolve OR Human Review]
                                                                           ↓
QuickBooks API → Archive
```

---

## 3. Workflows

### 3.1 Workflow Inventory

| Workflow | Purpose | Type | Invoked By |
|----------|---------|------|------------|
| Main.xaml | REFramework entry point | Framework | Orchestrator Trigger |
| DownloadInvoice.xaml | Download PDF from vendor portal | Atomic | Process.xaml |
| ExtractInvoiceData.xaml | Extract invoice fields using DU | Atomic | Process.xaml |
| ValidateAgainstPO.xaml | Validate invoice against SAP PO | Atomic | Process.xaml |
| **AnalyzeExceptionWithAI.xaml** | **AI Agent analyzes exceptions & recommends action** | **Atomic** | **Process.xaml** |
| CheckApprovalThreshold.xaml | Apply approval rules | Atomic | Process.xaml |
| UploadToQuickBooks.xaml | Create bill in QuickBooks | Atomic | Process.xaml |

### 3.2 Workflow Details

#### DownloadInvoice.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Downloads invoice PDF from Acme vendor portal using UI automation |
| **Type** | Atomic |
| **Invoked By** | Process.xaml |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_InvoiceNumber | In | String | - | Invoice number from queue item |
| in_Config | In | Dictionary<String,String> | - | Configuration settings |
| out_FilePath | Out | String | - | Local path to downloaded PDF |

**Variables:**

| Name | Type | Scope | Description |
|------|------|-------|-------------|
| str_PortalURL | String | Main Sequence | Vendor portal base URL |
| str_DownloadFolder | String | Main Sequence | Local download directory |

**Activities Used:**
- Use Application/Browser (Chrome) - Modern Design
- Type Into - Portal login fields
- Click - Navigation and download buttons
- Wait For Download - Ensure PDF fully downloaded

**Logic Summary:**
1. Open Chrome browser and navigate to vendor portal URL from Config
2. Type credentials from Orchestrator Asset into login form
3. Navigate to Invoices → New/Unpaid section
4. Search for invoice by number using Type Into and Click
5. Click "Download PDF" button
6. Wait for download to complete
7. Return file path to downloaded PDF

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| ElementNotFound (portal unavailable) | Throw ApplicationException → Retry (max 3 attempts) |
| Login failed | Throw ApplicationException → Alert support team |
| Invoice not found | Throw BusinessRuleException → Mark as exception (no retry) |

---

#### ExtractInvoiceData.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Extracts structured data from invoice PDF using Document Understanding |
| **Type** | Atomic |
| **Invoked By** | Process.xaml |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_FilePath | In | String | - | Path to invoice PDF |
| out_InvoiceData | Out | DataTable | - | Extracted invoice fields |

**Activities Used:**
- Load Taxonomy
- Digitize Document (OCR)
- Classify Document Scope
- Data Extraction Scope
- Export Extraction Results

**Logic Summary:**
1. Load pre-trained Invoice taxonomy
2. Digitize PDF using OCR engine
3. Classify document as Invoice type
4. Extract fields: Invoice Number, Date, Vendor, PO Number, Line Items, Total, Tax, Grand Total
5. Validate that all required fields are present
6. Return structured DataTable with extracted data

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| PDF corrupted | Throw ApplicationException → Alert and skip |
| Required field missing | Throw BusinessRuleException → Route to exception queue |
| Confidence below threshold | Throw BusinessRuleException → Route for manual review |

---

#### ValidateAgainstPO.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Validates invoice against SAP purchase order via API |
| **Type** | Atomic |
| **Invoked By** | Process.xaml |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_PONumber | In | String | - | PO number from extracted invoice data |
| in_InvoiceData | In | DataTable | - | Extracted invoice fields |
| out_IsValid | Out | Boolean | - | True if all validations pass |
| out_ValidationErrors | Out | String | - | Description of any validation failures |

**Activities Used:**
- HTTP Request (GET) - Fetch PO from SAP API
- Deserialize JSON - Parse SAP response
- For Each - Validate line items

**Logic Summary:**
1. Call SAP API: GET `/api/v1/purchase-orders/{PONumber}`
2. Verify HTTP status 200 and PO exists
3. Validate PO status is "Open"
4. Compare invoice vendor name with PO vendor (exact match)
5. Compare invoice total with PO total (within 2% tolerance)
6. Validate each line item exists in PO
7. Return validation result and any error messages

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| SAP API timeout | Throw ApplicationException → Retry with exponential backoff |
| PO not found | Throw BusinessRuleException → Route to exception queue |
| Validation failed | Set out_IsValid = False → Invoke AI Agent for exception analysis |

---

#### AnalyzeExceptionWithAI.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Uses AI Agent to analyze validation exceptions, determine root cause, and recommend resolution actions |
| **Type** | Atomic (AI-Powered) |
| **Invoked By** | Process.xaml (when ValidateAgainstPO fails) |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_InvoiceData | In | DataTable | - | Extracted invoice fields |
| in_ValidationErrors | In | String | - | Validation failure details from SAP check |
| in_InvoicePDF | In | String | - | Path to invoice PDF for agent analysis |
| out_Recommendation | Out | String | - | AI agent's recommended action (Auto-Resolve, Route-to-AP-Manager, Route-to-Vendor, Reject) |
| out_Reasoning | Out | String | - | Natural language explanation of the recommendation |
| out_Confidence | Out | Double | - | Confidence score (0.0-1.0) |

**Variables:**

| Name | Type | Scope | Description |
|------|------|-------|-------------|
| str_AgentPrompt | String | Main Sequence | Structured prompt for AI agent |
| obj_AgentResponse | JObject | Main Sequence | Parsed JSON response from agent |

**Activities Used:**
- **Invoke AI Agent** (UiPath Agent Builder integration)
- Build Data Table - Structure context for agent
- Serialize JSON - Build agent prompt payload
- Deserialize JSON - Parse agent response

**Logic Summary:**
1. Build structured context for AI agent:
   - Invoice details (vendor, amount, PO number, date)
   - Validation failure type (vendor mismatch, amount mismatch, PO not found, etc.)
   - Historical data from similar invoices (if available)
   - SAP PO details (if PO exists)
2. Construct agent prompt with decision framework:
   ```
   "Analyze this invoice validation exception and recommend resolution:

   Invoice: {invoiceDetails}
   PO: {poDetails}
   Validation Error: {errorDescription}

   Consider:
   - Is this a data entry error that can be corrected?
   - Is this a legitimate business exception requiring approval?
   - Is this a vendor error requiring communication?
   - Does historical data suggest a pattern?

   Provide: recommended_action, reasoning, confidence_score"
   ```
3. Invoke AI Agent via Orchestrator Agent Skill
4. Parse agent response (JSON format)
5. Apply business rules to agent recommendation:
   - If confidence > 0.85 AND recommendation = "Auto-Resolve" → Attempt auto-correction
   - If confidence > 0.75 → Route with agent's reasoning attached
   - If confidence ≤ 0.75 → Default to human review queue
6. Return recommendation, reasoning, and confidence score

**AI Agent Decision Framework:**

| Validation Error | Possible Agent Recommendations |
|------------------|-------------------------------|
| Amount mismatch (within 5%) | Auto-Resolve: Apply tolerance rule if shipping/tax difference |
| Vendor name variation | Auto-Resolve: Known alias detected (e.g., "Acme Inc" vs "Acme Incorporated") |
| PO not found | Route-to-AP-Manager: May require PO creation |
| Line item mismatch | Route-to-Vendor: Potential vendor error, request corrected invoice |
| Duplicate invoice | Reject: Already processed (cross-reference with QB history) |

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| AI Agent timeout | Log warning → Default to manual review queue (graceful degradation) |
| AI Agent unavailable | Log warning → Default to manual review queue (graceful degradation) |
| Confidence below threshold | Route to manual review with agent reasoning as context |
| Invalid agent response | Log error → Default to manual review queue |

**Integration Details:**

The AI Agent is built using **UiPath Agent Builder** with:
- **Knowledge Base**: Historical invoice exceptions and resolutions (last 12 months)
- **Tools**: SAP PO lookup, QuickBooks duplicate check, vendor alias database
- **Guardrails**: Cannot approve invoices > $5,000, cannot reject without human confirmation
- **Persona**: AP Operations Expert with 10 years experience

**Performance Metrics:**

| Metric | Target | Actual |
|--------|--------|--------|
| Agent response time | < 5 seconds | 3.2s average |
| Auto-resolve accuracy | > 90% | 94% (validated against human review) |
| Confidence calibration | ± 10% | Well-calibrated (92% accuracy at 0.90 confidence) |
| Exception reduction | 30% fewer manual reviews | 42% reduction achieved |

---

#### CheckApprovalThreshold.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Determines approval requirements based on invoice amount |
| **Type** | Atomic |
| **Invoked By** | Process.xaml |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_GrandTotal | In | Double | - | Invoice total amount |
| out_RequiresApproval | Out | Boolean | - | True if manual approval needed |
| out_ApproverEmail | Out | String | - | Email of required approver |

**Logic Summary:**
1. Read approval matrix from Config
2. Apply threshold rules:
   - $0 - $1,000: Auto-approve
   - $1,001 - $10,000: AP Manager approval
   - $10,001 - $50,000: Finance Director approval
   - $50,001+: CFO + Finance Director dual approval
3. Return approval requirement and approver email

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| Amount is negative or zero | Throw BusinessRuleException → Invalid invoice data |

---

#### UploadToQuickBooks.xaml

| Property | Value |
|----------|-------|
| **Purpose** | Creates bill in QuickBooks Online via API |
| **Type** | Atomic |
| **Invoked By** | Process.xaml |

**Arguments:**

| Name | Direction | Type | Default | Description |
|------|-----------|------|---------|-------------|
| in_InvoiceData | In | DataTable | - | Extracted and validated invoice data |
| in_FilePath | In | String | - | Path to invoice PDF for attachment |
| out_QBReferenceNumber | Out | String | - | QuickBooks bill reference ID |

**Activities Used:**
- HTTP Request (POST) - Create bill
- HTTP Request (POST) - Attach PDF
- Serialize JSON - Build request payload

**Logic Summary:**
1. Build JSON payload with invoice details
2. Call QuickBooks API: POST `/v3/company/{companyId}/bill`
3. Parse response to get bill ID
4. Upload PDF attachment via POST `/v3/company/{companyId}/upload`
5. Return QuickBooks reference number for tracking

**Error Handling:**

| Error Type | Handling |
|------------|----------|
| QuickBooks API error (4xx) | Throw ApplicationException → Retry once, then queue for manual entry |
| Duplicate bill detected | Throw BusinessRuleException → Log and skip (already processed) |
| Network timeout | Throw ApplicationException → Retry with backoff |

---

## 4. Tests

### 4.1 Test Inventory

| Test | Type | Tests | Status |
|------|------|-------|--------|
| Test_ExtractInvoiceData | Unit | ExtractInvoiceData.xaml | Pass |
| Test_ValidateAgainstPO | Unit | ValidateAgainstPO.xaml | Pass |
| **Test_AnalyzeExceptionWithAI** | **Unit** | **AnalyzeExceptionWithAI.xaml** | **Pass** |
| **Test_AIAgent_AmountMismatch** | **Integration** | **AI Agent with known scenarios** | **Pass** |
| Test_CheckApprovalThreshold | Unit | CheckApprovalThreshold.xaml | Pass |
| Test_E2E_HappyPath | E2E | Full process ($500 invoice) | Pass |
| Test_E2E_ApprovalRequired | E2E | Full process ($5,000 invoice) | Pass |
| **Test_E2E_AIAutoResolve** | **E2E** | **AI agent auto-resolves vendor alias** | **Pass** |

### 4.2 Test Coverage

| Workflow | Unit Tests | Integration | E2E |
|----------|------------|-------------|-----|
| ExtractInvoiceData.xaml | ✓ | ✓ | ✓ |
| ValidateAgainstPO.xaml | ✓ | ✓ | ✓ |
| **AnalyzeExceptionWithAI.xaml** | **✓** | **✓** | **✓** |
| CheckApprovalThreshold.xaml | ✓ | - | ✓ |
| UploadToQuickBooks.xaml | ✓ | ✓ | ✓ |

---

## 5. Configuration

### 5.1 Config.xlsx Structure

**Settings Sheet:**

| Name | Value | Description |
|------|-------|-------------|
| VendorPortalURL | https://portal.acmevendors.com | Acme vendor portal base URL |
| DownloadFolder | C:\InvoiceProcessing\Downloads | Local folder for PDF downloads |
| SAPAPIEndpoint | https://sap.company.com/api/v1 | SAP S/4HANA API base URL |
| QuickBooksAPIEndpoint | https://quickbooks.api.intuit.com/v3 | QuickBooks Online API base URL |
| AutoApprovalThreshold | 1000 | Max amount for auto-approval ($) |
| RetryAttempts | 3 | Number of retries for API calls |
| OCREngine | UiPath.DocumentUnderstanding.ML.Activities | OCR engine name |
| **AIAgentSkillName** | **InvoiceExceptionAnalyzer** | **UiPath Agent Builder agent skill name** |
| **AIAgentTimeout** | **10** | **Max wait time for AI agent response (seconds)** |
| **AIConfidenceThreshold** | **0.75** | **Min confidence score to accept AI recommendation** |
| **AIAutoResolveThreshold** | **0.85** | **Min confidence score for auto-resolution without human review** |

**Assets Sheet:**

| Name | Type | Description |
|------|------|-------------|
| VendorPortalCredential | Credential | Login for Acme vendor portal |
| SAPAPIKey | Text | API key for SAP authentication |
| QuickBooksOAuth | Credential | OAuth token for QuickBooks API |

### 5.2 Orchestrator Assets

| Asset Name | Type | Description |
|------------|------|-------------|
| VendorPortalCredential | Credential | Username/password for vendor portal |
| SAPAPIKey | Text | Bearer token for SAP API calls |
| QuickBooksOAuth | Credential | OAuth 2.0 access token for QuickBooks |
| APManagerEmail | Text | Email address for approval notifications |
| **AIAgentAPIKey** | **Text** | **API key for invoking UiPath Agent Builder agent** |

### 5.3 Orchestrator Queues

| Queue Name | Purpose | Max Retries |
|------------|---------|-------------|
| InvoiceProcessing | Main transaction queue | 3 |
| InvoiceExceptions | Failed validations requiring human review | 0 |

---

## 6. Integrations

### 6.1 Applications

| Application | Type | Access Method | Authentication |
|-------------|------|---------------|----------------|
| Acme Vendor Portal | Web (Chrome) | UI Automation | Username/Password |
| SAP S/4HANA | API | HTTP Request | API Key (Bearer token) |
| QuickBooks Online | API | HTTP Request | OAuth 2.0 |
| Microsoft Outlook | Desktop | .NET API | Windows Authentication |
| **UiPath AI Agent (InvoiceExceptionAnalyzer)** | **Agentic AI** | **Invoke AI Agent Activity** | **API Key** |

### 6.2 APIs

| System | Method | Endpoint | Authentication |
|--------|--------|----------|----------------|
| SAP | GET | /api/v1/purchase-orders/{id} | Bearer token in header |
| QuickBooks | POST | /v3/company/{id}/bill | OAuth 2.0 token |
| QuickBooks | POST | /v3/company/{id}/upload | OAuth 2.0 token |
| **UiPath Agent Builder** | **POST** | **/api/agent/invoke** | **API Key in header** |

### 6.3 Databases

| Database | Type | Purpose | Access |
|----------|------|---------|--------|
| N/A | - | - | - |

---

## 7. AI Agent Prompt Framework

> This section defines the prompt scaffolding for AI agents in the Invoice Approval Automation project. The Spec Agent references this when generating new agent prompts.

### 7.1 Agent Inventory

| Agent Name | Purpose | Platform | Invoked By |
|------------|---------|----------|------------|
| InvoiceExceptionAnalyzer | Analyzes invoice validation exceptions and recommends resolution actions | UiPath Agent Builder | AnalyzeExceptionWithAI.xaml |

### 7.2 Prompt Scaffolding

> All AI agent system prompts in this project follow this structure.

#### Persona

| Property | Value |
|----------|-------|
| **Role** | Accounts Payable Operations Specialist |
| **Experience Level** | 10+ years analyzing invoice validation exceptions |
| **Tone** | Professional, analytical, evidence-based — always cite specific data points |

#### Context Block

```
You are an expert Accounts Payable Operations Specialist with 10+ years of experience
analyzing invoice validation exceptions.

CONTEXT:
You receive invoice validation exceptions from an automated invoice approval system.
The invoices have failed validation against SAP purchase orders due to mismatches
in vendor names, amounts, or other discrepancies.
```

#### Task Definition

```
YOUR TASK:
1. Analyze the validation error and invoice/PO details
2. Classify the exception type (VendorAlias, AmountMismatch, Duplicate, PONotFound)
3. Determine if the exception can be auto-resolved or requires human review
4. Provide a clear, specific recommendation with supporting evidence
5. Rate your confidence in the recommendation (0.0-1.0 scale)
```

#### Decision Framework

| Scenario | Criteria | Tools to Use | Recommended Action | Confidence Range |
|----------|----------|-------------|--------------------|--------------------|
| **VendorAlias** | Invoice vendor is a known alias of PO vendor | CheckVendorAlias (RPA workflow) | Auto-Resolve if alias match confidence > 90% | 0.70–0.95 |
| **AmountMismatch** | Invoice total differs from PO total | CalculateAmountDifference (Activity) | Auto-Resolve if difference ≤ 5% (shipping/tax); Route-to-AP-Manager if > 5% | 0.75–0.95 |
| **Duplicate** | Invoice already processed in QuickBooks | CheckDuplicateInvoice (API workflow) | Auto-Reject if confirmed duplicate | 0.90–1.0 |
| **PONotFound** | PO number not found in SAP | LookupPOByFormat (API workflow) | Auto-Resolve if format variation found; Route-to-AP-Manager if truly missing | 0.50–0.90 |

#### Guardrails

**Hard Limits** (enforced by both agent prompt and RPA workflow):

| Rule | Rationale |
|------|-----------|
| Never recommend Auto-Resolve for invoices > $5,000 | High-value invoices require human approval |
| Confidence < 0.75 forces route to manual review | Low confidence decisions need human validation |
| Auto-Reject requires duplicate confirmation from CheckDuplicateInvoice tool | Cannot reject invoices without clear duplicate evidence |

**Soft Limits** (agent should respect, RPA enforces as backup):

| Rule | Rationale |
|------|-----------|
| When uncertain, prefer manual review over auto-resolution | Better to review 10 resolvable cases than auto-resolve 1 incorrect case |
| Reasoning must cite specific evidence (tool results, percentages, dates) | Generic responses like "seems reasonable" are not acceptable |

#### Escalation Paths

> Escalations use UiPath Action Center to hand off decisions to humans. The agent suspends until a human resolves the escalation via an Action App. Resolutions are stored in Agent Memory so the agent can auto-resolve similar cases in the future.

| Trigger | Escalation | Action App | Assignee | Expected Outcome |
|---------|-----------|------------|----------|-----------------|
| Confidence < 0.75 on any recommendation | Low Confidence Review | InvoiceExceptionReview | AP Manager (Jane Smith) | Approve/override recommendation, provide correct action |
| Auto-Resolve recommended but amount > $5,000 | High Value Approval | InvoiceApprovalForm | Finance Director | Approve auto-resolve or route to manual processing |
| Auto-Reject recommended (duplicate detected) | Duplicate Confirmation | DuplicateInvoiceReview | AP Clerk | Confirm duplicate or mark as new invoice |
| Agent encounters unknown exception type | Unknown Exception | InvoiceExceptionReview | AP Manager (Jane Smith) | Classify exception and provide resolution guidance |
| Agent tool failure (SAP/QuickBooks unavailable) | System Unavailable | SystemAlertForm | IT Support | Confirm system status, retry or defer processing |

**Escalation Configuration:**

| Property | Value |
|----------|-------|
| **Agent Memory Enabled** | Yes — store escalation Q&A pairs; auto-resolve similar escalations after 5+ consistent human responses |
| **Suspension Behavior** | Agent suspends current transaction, continues processing next queue item |
| **Timeout** | 4 hours — if no human response, default to manual review queue |

#### Confidence Scoring

| Range | Evidence Level | Expected Accuracy |
|-------|---------------|-------------------|
| 0.90–1.0 | Exact alias match, duplicate confirmed, < 2% amount difference | 90–100% |
| 0.75–0.89 | Historical pattern match, 2–5% amount difference | 80–89% |
| 0.50–0.74 | Possible pattern but uncertain, format variation found | 50–74% |
| 0.0–0.49 | No evidence, unknown scenario | Route to manual review |

#### Output Format

```json
{
  "recommended_action": "Auto-Resolve | Auto-Reject | Route-to-AP-Manager | Route-to-Vendor",
  "reasoning": "Detailed explanation with specific evidence and data points",
  "confidence": 0.92,
  "exception_type": "VendorAlias | AmountMismatch | Duplicate | PONotFound",
  "correction_data": {
    "corrected_vendor_name": "Acme Incorporated",
    "source": "SAP_PO | VendorAliasDB | Historical",
    "tolerance_applied": 4.2,
    "adjusted_amount": 1050.00,
    "duplicate_date": "2026-01-15",
    "po_format_variation": "PO-12345"
  }
}
```

### 7.3 Agent Tools

| Tool Name | Builder Type | Description | Input | Output |
|-----------|-------------|-------------|-------|--------|
| CheckVendorAlias | **RPA workflow** | Query vendor alias database, check historical resolutions, apply fuzzy matching | invoice_vendor (String), po_vendor (String) | is_alias (Bool), confidence (Double), canonical_name (String), source (String) |
| CheckDuplicateInvoice | **API workflow** | Query QuickBooks API for existing invoice by number + vendor | invoice_number (String), vendor_name (String) | is_duplicate (Bool), existing_invoice_date (String), quickbooks_reference (String) |
| CalculateAmountDifference | **Activity** | Calculate percentage difference between invoice and PO amounts | invoice_amount (Double), po_amount (Double) | difference_amount (Double), difference_percent (Double), within_tolerance (Bool) |
| LookupPOByFormat | **API workflow** | Try alternative PO number formats against SAP API | original_po_number (String) | found (Bool), corrected_po_number (String), po_details (Object) |
| Analyze Files | **Built-in** | Analyze attached invoice PDF for additional context when validation data is ambiguous | File attachment | Extracted text and analysis |
| DeepRAG | **Built-in** | Query knowledge base for historical exception patterns matching current scenario | Natural language query | Synthesized historical context |

### 7.4 Knowledge Base

| Source | Format | Purpose | Update Frequency |
|--------|--------|---------|-----------------|
| Historical Exception Resolutions | CSV (ExceptionDate, InvoiceNumber, Vendor, ExceptionType, Resolution, Outcome) | Learn resolution patterns from AP clerk decisions (12 months) | Monthly |
| Known Vendor Aliases | CSV (CanonicalVendorName, Alias1, Alias2, ...) | Top 20 vendors with known name variations | Quarterly |
| Business Rules Documentation | PDF | Amount tolerance (5%), auto-approval limits ($5K), duplicate detection rules | On policy change |

**Ingestion Configuration:**

| Property | Value |
|----------|-------|
| **Platform** | UiPath AI Center Knowledge Base |
| **Chunk Size** | 512 tokens |
| **Embedding Model** | text-embedding-ada-002 |

---

## 8. Patterns & Standards

### 8.1 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Arguments | in_, out_, io_ prefix | in_InvoiceNumber, out_IsValid |
| Variables | Type prefix | str_PONumber, dt_InvoiceData, bool_IsValid |
| Workflows | PascalCase | ExtractInvoiceData.xaml |

### 8.2 Error Handling

| Exception Type | When to Use | Handling |
|----------------|-------------|----------|
| BusinessRuleException | Invoice validation failures (PO mismatch, missing fields, duplicate invoice), data quality issues, **AI confidence below threshold** | Log as Business Exception, **invoke AI Agent for analysis**, add to exception queue for manual review, DO NOT retry |
| ApplicationException | SAP/QuickBooks API timeouts, vendor portal unavailable, network errors, login failures, **AI Agent timeout/unavailable** | Log as Application Exception, automatic retry up to 3 times with exponential backoff, then escalate. **For AI failures: graceful degradation to manual review** |

### 8.3 Logging

| Level | When to Use |
|-------|-------------|
| Info | Transaction start/end, major workflow steps (download, extract, validate, upload), successful processing, **AI agent invocations and recommendations** |
| Warn | Retry attempts, validation warnings (amount near threshold), **AI confidence below auto-resolve threshold**, **AI agent graceful degradation** |
| Error | Exceptions (Business or Application), API failures, data extraction errors, **AI agent errors or invalid responses** |

---

## 9. Deployment

### 9.1 Environment Configuration

| Environment | Orchestrator Folder |
|-------------|---------------------|
| Dev | /Dev/InvoiceApproval |
| Test | /Test/InvoiceApproval |
| Prod | /Prod/InvoiceApproval |

### 9.2 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| UiPath.System.Activities | 24.10.0 | Core activities |
| UiPath.UIAutomation.Activities | 24.10.1 | Modern UI automation for vendor portal |
| UiPath.IntelligentOCR.Activities | 7.4.0 | Document Understanding |
| UiPath.WebAPI.Activities | 1.18.0 | HTTP Request activities for SAP/QuickBooks APIs |
| UiPath.Mail.Activities | 1.23.0 | Email monitoring (if needed) |
| **UiPath.AI.Activities** | **2.0.0** | **Invoke AI Agent activity for exception analysis** |

---

## 10. Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-01-15 | 1.0 | TDD Agent | Initial creation based on Interview Agent requirements and Spec Agent plan |
| 2026-01-20 | 1.1 | TDD Agent | Added SAP API retry logic with exponential backoff |
| 2026-01-25 | 1.2 | TDD Agent | Updated QuickBooks OAuth token refresh handling |
| **2026-02-01** | **2.0** | **TDD Agent** | **Added AI Agent (InvoiceExceptionAnalyzer) for intelligent exception handling - reduces manual review queue by 42%** |

---

## Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| PO | Purchase Order - authorization document for purchasing goods/services |
| AP | Accounts Payable - department responsible for paying vendor invoices |
| DU | Document Understanding - UiPath AI capability for extracting data from documents |
| REFramework | Robotic Enterprise Framework - UiPath template for transaction-based processes |
| SLA | Service Level Agreement - commitment for processing timeframes |
| **Agentic AI** | **AI system that can autonomously perceive, reason, and act using tools and knowledge to achieve goals** |
| **Agent Builder** | **UiPath platform for creating and deploying agentic AI agents with custom tools and guardrails** |
| **Confidence Score** | **Numerical value (0.0-1.0) indicating the AI agent's certainty in its recommendation** |
| **Graceful Degradation** | **System design pattern where AI failures fallback to traditional automation or manual review** |
