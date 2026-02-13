# Technical Design Document: Invoice Approval Automation

**Version:** 1.0
**Status:** Production

---

## 1. Project Overview

### 1.1 Business Process

| Property | Value |
|----------|-------|
| **Process Name** | Invoice Approval Automation |
| **Business Owner** | Jane Smith, Accounts Payable Department |
| **Description** | Automates the invoice approval process from email receipt through QuickBooks upload, including PO validation and approval routing |
| **Trigger** | Orchestrator Queue (triggered by email arrival) |
| **Frequency** | Continuous (50-100 invoices/day) |

### 1.2 Scope

**In Scope:**
- Monitor shared AP inbox for invoice emails
- Download invoices from vendor portal
- Extract invoice data using Document Understanding
- Validate invoices against SAP purchase orders via API
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
│   ├── [CheckApprovalThreshold.xaml]
│   └── [UploadToQuickBooks.xaml]
└── [Framework/CloseAllApplications.xaml]
```

### 2.3 Data Flow

```
Email Inbox → Queue Item → Download PDF → Document Understanding → SAP API Validation → QuickBooks API → Archive
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
| Validation failed | Set out_IsValid = False → Route to manual review (not an exception, expected path) |

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
| Test_CheckApprovalThreshold | Unit | CheckApprovalThreshold.xaml | Pass |
| Test_E2E_HappyPath | E2E | Full process ($500 invoice) | Pass |
| Test_E2E_ApprovalRequired | E2E | Full process ($5,000 invoice) | Pass |

### 4.2 Test Coverage

| Workflow | Unit Tests | Integration | E2E |
|----------|------------|-------------|-----|
| ExtractInvoiceData.xaml | ✓ | ✓ | ✓ |
| ValidateAgainstPO.xaml | ✓ | ✓ | ✓ |
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

### 6.2 APIs

| System | Method | Endpoint | Authentication |
|--------|--------|----------|----------------|
| SAP | GET | /api/v1/purchase-orders/{id} | Bearer token in header |
| QuickBooks | POST | /v3/company/{id}/bill | OAuth 2.0 token |
| QuickBooks | POST | /v3/company/{id}/upload | OAuth 2.0 token |

### 6.3 Databases

| Database | Type | Purpose | Access |
|----------|------|---------|--------|
| N/A | - | - | - |

---

## 7. Patterns & Standards

### 7.1 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Arguments | in_, out_, io_ prefix | in_InvoiceNumber, out_IsValid |
| Variables | Type prefix | str_PONumber, dt_InvoiceData, bool_IsValid |
| Workflows | PascalCase | ExtractInvoiceData.xaml |

### 7.2 Error Handling

| Exception Type | When to Use | Handling |
|----------------|-------------|----------|
| BusinessRuleException | Invoice validation failures (PO mismatch, missing fields, duplicate invoice), data quality issues, confidence below threshold | Log as Business Exception, add to exception queue for manual review, DO NOT retry |
| ApplicationException | SAP/QuickBooks API timeouts, vendor portal unavailable, network errors, login failures | Log as Application Exception, automatic retry up to 3 times with exponential backoff, then escalate |

### 7.3 Logging

| Level | When to Use |
|-------|-------------|
| Info | Transaction start/end, major workflow steps (download, extract, validate, upload), successful processing |
| Warn | Retry attempts, validation warnings (amount near threshold) |
| Error | Exceptions (Business or Application), API failures, data extraction errors |

---

## 8. Deployment

### 8.1 Environment Configuration

| Environment | Orchestrator Folder |
|-------------|---------------------|
| Dev | /Dev/InvoiceApproval |
| Test | /Test/InvoiceApproval |
| Prod | /Prod/InvoiceApproval |

### 8.2 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| UiPath.System.Activities | 24.10.0 | Core activities |
| UiPath.UIAutomation.Activities | 24.10.1 | Modern UI automation for vendor portal |
| UiPath.IntelligentOCR.Activities | 7.4.0 | Document Understanding |
| UiPath.WebAPI.Activities | 1.18.0 | HTTP Request activities for SAP/QuickBooks APIs |
| UiPath.Mail.Activities | 1.23.0 | Email monitoring (if needed) |

---

## 9. Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-01-15 | 1.0 | TDD Agent | Initial creation based on Interview Agent requirements and Spec Agent plan |
| 2026-01-20 | 1.1 | TDD Agent | Added SAP API retry logic with exponential backoff |
| 2026-01-25 | 1.2 | TDD Agent | Updated QuickBooks OAuth token refresh handling |

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
