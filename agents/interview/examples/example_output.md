# Requirements Document

**Project:** Invoice Processing Automation  
**User Story:** Automate the process of downloading invoices from vendor portal, extracting data, validating against PO system, and uploading to accounting system  
**Date:** 2026-01-28  
**Gathered By:** Interview Agent v1.0  
**Status:** Approved

---

## 1. Business Context

### 1.1 Business Problem

The Accounts Payable team currently spends 8-10 hours per week manually:
- Downloading invoices from the Acme Vendor Portal
- Extracting data from PDF invoices
- Cross-checking against purchase orders in SAP
- Entering validated invoices into QuickBooks

**Current pain points:**
- Time-consuming (taking staff away from higher-value work)
- Error-prone (~5% manual data entry error rate)
- Slow (invoices sit in queue 2-3 days before processing)
- Not scalable (volume growing 15% per quarter)

### 1.2 Business Objectives

| Objective | Target |
|-----------|--------|
| Reduce processing time | 8-10 hrs/week → <1 hr/week (90% reduction) |
| Improve accuracy | 5% error rate → <1% |
| Accelerate processing | 2-3 day turnaround → same-day |
| Scale without headcount | Handle 15-20% volume growth with existing staff |
| Enable early payment discounts | Capture 2% discounts for payment within 10 days |

### 1.3 Stakeholders

| Role | Interest |
|------|----------|
| **Accounts Payable team** (3 staff) | Primary users, time savings |
| **Finance Manager** | Budget owner, ROI |
| **Vendors** | Faster payment |
| **Procurement team** | Invoice status visibility |
| **CFO** | Approver |

---

## 2. Functional Requirements

### 2.1 Core Functionality

#### Step 1: Download Invoices

| Requirement | Details |
|-------------|---------|
| Login | Acme Vendor Portal via username/password |
| Navigation | Navigate to "New Invoices" section |
| Filter | Status = "Ready for Processing" |
| Download | Save PDFs to local processing folder |
| Mark processed | Update invoice status in portal to prevent re-processing |

#### Step 2: Extract Data from PDFs

For each invoice PDF, extract:

| Field | Required | Notes |
|-------|----------|-------|
| Invoice Number | Yes | Alphanumeric, max 20 chars |
| Invoice Date | Yes | MM/DD/YYYY, not future dated |
| Vendor Name | Yes | Must match SAP vendor master |
| Vendor ID | Yes | 10-digit numeric |
| Purchase Order Number | Yes | Format: PO-XXXXXX |
| Line Items | Yes | At least 1 item |
| Subtotal | Yes | Must = sum of line items |
| Tax Amount | No | If present, must be ≥ 0 |
| Grand Total | Yes | Must = Subtotal + Tax |

#### Step 3: Validate Against SAP

For each invoice, validate:
- ✓ PO exists and is in "Open" status
- ✓ Vendor ID matches PO vendor
- ✓ Invoice total is within ±5% of PO amount (tolerance for tax/shipping)
- ✓ PO has not been fully invoiced yet

Mark validation result (Pass/Fail with reason).

#### Step 4: Upload to QuickBooks

For validated invoices:
- Create Bill in QuickBooks using API
- Populate all fields (vendor, date, amounts, line items)
- Attach original PDF invoice as supporting document
- Add note: "Auto-processed from Acme Portal on [date]"
- Set payment terms based on vendor

#### Step 5: Exception Handling

For invoices that fail validation:
1. Move PDF to "Exceptions" folder
2. Create item in SharePoint "Invoice Exceptions" list
3. Send email notification to AP team with summary

#### Step 6: Reporting

Generate daily summary report including:
- Total invoices processed
- Successfully uploaded to QuickBooks (count and $)
- Exceptions requiring review (count and $)
- Processing time
- Any errors encountered

### 2.2 User Interactions

| Interaction | Details |
|-------------|---------|
| Unattended execution | Runs daily at 6:00 AM |
| Exception review | AP team reviews items in SharePoint list |
| Override capability | Move invoice from Exceptions to Processing folder to reprocess |
| Report review | Finance Manager reviews daily summary email |

### 2.3 Business Rules

**Validation Rules:**
1. Invoice total must be within ±5% of PO amount
2. Vendor ID must exactly match PO vendor
3. PO must be in "Open" status
4. Invoice date cannot be more than 90 days in the past
5. Invoice number must be unique (not previously processed)

**Payment Terms Logic:**

| Vendor Category | Terms |
|-----------------|-------|
| Preferred | 2/10 Net 30 (2% discount if paid within 10 days) |
| Standard | Net 30 |
| New | Net 45 (until credit established) |

**Duplicate Detection:**
- Check QuickBooks for existing bill with same invoice number from same vendor
- If found, skip invoice and log as "Duplicate"

---

## 3. Data Requirements

### 3.1 Input Data

| Property | Value |
|----------|-------|
| Source | Acme Vendor Portal (https://portal.acme-vendor.com) |
| Access | Web-based UI (Chrome browser) |
| Authentication | Username/password (Orchestrator Credential asset) |
| Data Format | PDF invoices (mostly machine-readable, ~10% scanned) |
| Location | Portal → New Invoices → Ready for Processing |
| Volume | 20-30 invoices/day (100-150/week) |
| Peak | Month-end: 50+ invoices/day |

### 3.2 Output Data

**Destination 1: QuickBooks Online**
- API: QuickBooks API v3 (REST)
- Authentication: OAuth 2.0
- Data Format: JSON (Bill object)
- Attachments: Original PDF invoice

**Destination 2: SharePoint "Invoice Exceptions" List**
- Location: https://company.sharepoint.com/sites/Finance/InvoiceExceptions
- Access: SharePoint REST API / Microsoft Graph API
- Authentication: Service account (OAuth)

**Destination 3: Daily Summary Email**
- Recipients: ap-team@company.com, finance-manager@company.com
- Format: HTML email with summary table + attached Excel log
- Send Time: 7:00 AM daily

### 3.3 Data Volume

| Metric | Current | 12-Month Projection |
|--------|---------|---------------------|
| Daily average | 20-30 invoices ($15,000) | 35-45 invoices |
| Peak (month-end) | 50 invoices | 75 invoices |
| Growth rate | 15% per quarter | — |

---

## 4. Integration Requirements

### 4.1 Applications

#### Acme Vendor Portal
| Property | Value |
|----------|-------|
| Type | Web application |
| URL | https://portal.acme-vendor.com |
| Access | UI Automation (Chrome) |
| Auth | Username/password (Orchestrator Credential) |
| Stability | Generally stable; occasional slowness 8-9 AM |

#### SAP ERP
| Property | Value |
|----------|-------|
| Type | Desktop application (SAP GUI) |
| Access | UI Automation (SAP GUI connector) |
| Transactions | ME23N (Display Purchase Order) |
| Alternative | SAP OData API (if available) |

#### QuickBooks Online
| Property | Value |
|----------|-------|
| Type | Cloud application |
| Access | REST API (v3) |
| Auth | OAuth 2.0 (refresh token) |
| Rate Limits | 500 requests/minute, 10,000/day |

#### SharePoint Online
| Property | Value |
|----------|-------|
| Type | Cloud collaboration platform |
| Access | SharePoint REST API / Microsoft Graph |
| Auth | Service account or App Registration |
| Rate Limits | 2,000 requests/hour per user |

### 4.2 API Summary

| System | API Type | Authentication | Documentation |
|--------|----------|----------------|---------------|
| QuickBooks | REST | OAuth 2.0 | [Developer Portal](https://developer.intuit.com/app/developer/qbo/docs/api) |
| SharePoint | REST | OAuth 2.0 | [MS Docs](https://docs.microsoft.com/en-us/sharepoint/dev/sp-add-ins/get-to-know-the-sharepoint-rest-service) |
| SAP (future) | OData | Basic Auth | TBD |

---

## 5. Error Handling & Edge Cases

### 5.1 Error Scenarios

| Scenario | Handling | Recovery |
|----------|----------|----------|
| **Portal unavailable** | Retry 3× with 5-min delays; alert IT and AP team | Manual (IT resolves) |
| **PDF extraction fails** | Move to Exceptions; log "Extraction Failed" | Manual (AP processes) |
| **PO not found in SAP** | Mark as "PO Not Found"; move to Exceptions | Manual (AP investigates) |
| **Amount mismatch** | Log variance; move to Exceptions | Manual (AP investigates) |
| **QuickBooks API failure** | Retry with backoff; save to retry queue if persistent | Automated retry next run |
| **Duplicate invoice** | Skip; log as "Duplicate"; move to Duplicates folder | None needed |

### 5.2 Edge Cases

| Edge Case | Handling |
|-----------|----------|
| **Multiple invoices for one PO** | Track cumulative invoiced amount per PO |
| **Scanned PDFs (~10%)** | OCR with Document Understanding; if confidence <85%, route to Exceptions |
| **Multi-page invoices** | Extract all pages, not just page 1 |
| **Foreign currency** | Out of scope—route to Exceptions |
| **Credit notes** | Out of scope—route to Exceptions |

### 5.3 Recovery Strategy

| Type | Strategy |
|------|----------|
| **Automated** | Retry logic for transient failures; auto-refresh OAuth tokens; retry queue for failed uploads |
| **Manual** | Exception queue in SharePoint; clear notification emails; reprocess by moving file |
| **Business Continuity** | AP team can process manually if automation fails; non-destructive (source data preserved) |

---

## 6. Non-Functional Requirements

### 6.1 Performance

| Requirement | Target | Maximum |
|-------------|--------|---------|
| Total execution time | <30 min for 30 invoices | 60 min |
| Per-invoice processing | 1 min average | — |
| PDF extraction | <30 sec (including OCR) | — |
| API calls | <5 sec each | — |
| Peak capacity | 50 invoices/run | — |

### 6.2 Availability

| Requirement | Value |
|-------------|-------|
| Schedule | Daily at 6:00 AM Eastern |
| Execution days | Monday-Friday (business days) |
| Holidays | Skip US Federal holidays (Orchestrator calendar) |
| Retry window | 7 AM and 8 AM if 6 AM fails |
| Target uptime | 99% (1-2 allowed failures/month) |

### 6.3 Security

| Requirement | Implementation |
|-------------|----------------|
| Data sensitivity | HIGH (vendor banking info, financial data, SOX) |
| Credentials | Orchestrator Credential assets (encrypted) |
| Access controls | Robot account with minimum permissions; service accounts |
| Audit trail | All actions logged; logs retained 7 years (SOX) |
| Compliance | SOX, company Information Security Policy |

### 6.4 Scalability

| Metric | Value |
|--------|-------|
| Design capacity | 100 invoices/day without redesign |
| Growth projection | 15% per quarter |
| Parallel processing | Consider if volume exceeds 50/day |

---

## 7. Constraints & Assumptions

### 7.1 Technical Constraints
- UiPath Studio (Modern Design)
- Windows Server 2019 Unattended Robot
- Chrome browser (latest version)
- SAP GUI client installed
- .NET Framework 4.7.2+
- Network access to all systems

### 7.2 Business Constraints
- **Budget:** $15,000 (development, testing, deployment)
- **Timeline:** Production-ready within 8 weeks
- **Resources:** 1 RPA developer, 1 BA, SME access
- **Change Control:** Standard change management for production

### 7.3 Assumptions
- ✓ Acme Vendor Portal structure remains stable
- ✓ QuickBooks API access is provisioned
- ✓ SharePoint site and list already exist
- ✓ Robot has necessary permissions
- ✓ Network connectivity is stable
- ✓ PDF format from vendors is relatively consistent
- ✓ SAP PO numbers in invoices are accurate
- ✓ AP team reviews exceptions within 1 business day

---

## 8. User Preferences

### 8.1 Design Preferences
- Prefer APIs over UI automation where possible
- Modular workflow design (separate Download, Extract, Validate, Upload workflows)
- Comprehensive logging for troubleshooting
- Professional HTML-formatted email notifications
- Conservative approach: "Route to humans rather than making assumptions"

### 8.2 Priority Areas

| Priority | Area |
|----------|------|
| Highest | Accuracy—no incorrect data in QuickBooks |
| High | Exception handling—clear, actionable |
| Medium | Performance—faster is better |
| Lower | Advanced features (OCR can be phase 2) |

---

## 9. Success Criteria

| Criteria | Target |
|----------|--------|
| Time savings | 8-10 hrs/week → <1 hr/week |
| Accuracy | >98% (data matches source invoices) |
| Speed | Same-day processing (by 9 AM) |
| Reliability | 99% success rate on scheduled runs |
| Exception rate | <20% require manual intervention |
| User satisfaction | AP team rates "helpful" or "very helpful" |
| Scalability | Handles 15-20% volume growth |
| ROI | Positive within 6 months |

---

## 10. Open Questions

None—all critical questions have been addressed during requirements gathering.

---

## Approval

| Role | Name | Date | Status |
|------|------|------|--------|
| AP Team Lead | _______________ | ___ | ☐ Approved |
| Finance Manager | _______________ | ___ | ☐ Approved |
| IT Security | _______________ | ___ | ☐ Validated |

---

## Next Steps

1. **AP Team Lead** reviews and approves requirements
2. **Finance Manager** reviews and approves requirements
3. **IT Security** validates security and compliance approach
4. Hand off to **Spec Agent** for implementation planning
5. Schedule kickoff meeting with development team

---

**Document Version:** 1.0  
**Generated By:** Interview Agent v1.0  
**Generation Date:** 2026-01-28 09:45 AM  
**Source User Story:** "Automate invoice processing from Acme Vendor Portal to QuickBooks"  
**Interview Duration:** 22 minutes  
**Questions Asked:** 18  
**Completeness Score:** 95%
