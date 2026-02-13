# Test Scenarios: AI-Powered Exception Analysis

**Project:** Invoice Approval Automation v2.0
**Story ID:** IAUTO-247
**Date:** February 2, 2026
**Author:** Spec Agent

---

## Table of Contents
1. [Unit Tests](#1-unit-tests)
2. [Integration Tests](#2-integration-tests)
3. [End-to-End Tests](#3-end-to-end-tests)
4. [Shadow Mode Validation](#4-shadow-mode-validation)
5. [Performance Tests](#5-performance-tests)

---

## 1. Unit Tests

**Purpose:** Test AnalyzeExceptionWithAI.xaml workflow logic with mocked AI responses

**Prerequisites:**
- Workflow published to Dev environment
- Mock AI agent responses configured

---

### UT-1: High Confidence Auto-Resolve (Vendor Alias)

**Test Data:**
```json
Mock AI Response:
{
  "recommended_action": "Auto-Resolve",
  "reasoning": "Vendor 'Acme Inc' is a known alias for 'Acme Incorporated' (PO vendor). Confirmed via VendorAliasDB with 95% historical match rate.",
  "confidence": 0.92,
  "exception_type": "VendorAlias",
  "correction_data": {
    "corrected_vendor_name": "Acme Incorporated",
    "source": "VendorAliasDB"
  }
}

Input Arguments:
- in_InvoiceData: Invoice with vendor "Acme Inc", amount $800
- in_ValidationErrors: "Vendor mismatch: Acme Inc != Acme Incorporated"
- in_POData: PO with vendor "Acme Incorporated", amount $800
```

**Expected Result:**
- ✓ out_RecommendedAction = "Auto-Resolve"
- ✓ out_Confidence = 0.92
- ✓ out_CorrectionData contains corrected vendor name
- ✓ Guardrails pass (amount ≤ $5K, confidence ≥ 0.85)
- ✓ Log message: "Guardrails passed, proceeding with auto-resolution"

---

### UT-2: Amount Guardrail Blocks Auto-Resolve

**Test Data:**
```json
Mock AI Response:
{
  "recommended_action": "Auto-Resolve",
  "reasoning": "Vendor alias confirmed",
  "confidence": 0.95,
  "exception_type": "VendorAlias",
  "correction_data": {...}
}

Input Arguments:
- in_InvoiceData: Invoice with amount $6,500 (exceeds $5K limit)
```

**Expected Result:**
- ✓ out_RecommendedAction overridden to "Route-to-AP-Manager"
- ✓ out_Reasoning appended with "[Auto-resolution blocked: Invoice amount exceeds $5,000 limit]"
- ✓ Log warning: "Auto-resolution blocked: Invoice amount $6,500 exceeds limit $5,000"

---

### UT-3: Low Confidence Routes to Manual Review

**Test Data:**
```json
Mock AI Response:
{
  "recommended_action": "Auto-Resolve",
  "confidence": 0.72,  // Below 0.75 threshold
  "exception_type": "AmountMismatch"
}
```

**Expected Result:**
- ✓ out_RecommendedAction overridden to "Route-to-AP-Manager"
- ✓ Log warning: "Confidence 72% below threshold 85%, routing to manual review"

---

### UT-4: AI Agent Timeout (Graceful Degradation)

**Test Data:**
- Mock AI agent with 10-second delay (exceeds 5s timeout)

**Expected Result:**
- ✓ CATCH block triggered after 5 seconds
- ✓ out_RecommendedAction = "Route-to-AP-Manager"
- ✓ out_Reasoning = "AI agent unavailable. Manual review required. Original error: ..."
- ✓ out_Confidence = 0.0
- ✓ Log warning: "AI Agent failed: timeout. Graceful degradation to manual review."
- ✓ Workflow completes successfully (does not throw exception)

---

### UT-5: Invalid AI Response JSON

**Test Data:**
```json
Mock AI Response (malformed JSON):
{
  "recommended_action": "Auto-Resolve",
  // Missing required fields: reasoning, confidence, exception_type
}
```

**Expected Result:**
- ✓ JSON parsing succeeds, but field extraction fails
- ✓ CATCH block triggered (NullReferenceException or similar)
- ✓ Graceful degradation to manual review
- ✓ Log warning: "AI Agent failed: [error details]. Graceful degradation to manual review."

---

### UT-6: Auto-Reject Duplicate Invoice

**Test Data:**
```json
Mock AI Response:
{
  "recommended_action": "Auto-Reject",
  "reasoning": "Duplicate invoice detected: INV-9999 already processed on 2026-01-15. QuickBooks reference: BILL-5678.",
  "confidence": 0.98,
  "exception_type": "Duplicate",
  "correction_data": {
    "duplicate_date": "2026-01-15",
    "quickbooks_reference": "BILL-5678"
  }
}
```

**Expected Result:**
- ✓ out_RecommendedAction = "Auto-Reject"
- ✓ out_Reasoning contains specific duplicate detection details
- ✓ out_Confidence = 0.98
- ✓ Workflow completes successfully

---

## 2. Integration Tests

**Purpose:** Test with real AI agent and known scenarios

**Prerequisites:**
- InvoiceExceptionAnalyzer agent deployed to Dev
- Test vendor alias database populated
- Test QuickBooks instance with known invoices

---

### IT-1: Vendor Alias - Known Alias (Acme Inc)

**Test Invoice:**
```
Invoice Number: INV-TEST-001
Vendor: Acme Inc
Amount: $1,200
PO Number: PO-5678
```

**Test PO (from SAP):**
```
PO Number: PO-5678
Vendor: Acme Incorporated
Amount: $1,200
```

**Vendor Alias DB Entry:**
```
Canonical: "Acme Incorporated"
Aliases: ["Acme Inc", "Acme Corp", "ACME"]
```

**Expected AI Response:**
- recommended_action: "Auto-Resolve"
- confidence: ≥ 0.90
- exception_type: "VendorAlias"
- correction_data.corrected_vendor_name: "Acme Incorporated"
- reasoning: Should mention "known alias" or "VendorAliasDB"

**Expected Workflow Behavior:**
- Guardrails pass (amount ≤ $5K, confidence ≥ 0.85)
- Vendor name corrected to "Acme Incorporated"
- Invoice proceeds to UploadToQuickBooks

**Validation:**
- ✓ AI agent returns expected response
- ✓ Workflow applies correction
- ✓ Logged: "AI auto-resolving exception: Vendor 'Acme Inc' is a known alias..."

---

### IT-2: Vendor Alias - Historical Pattern (Not in DB)

**Test Invoice:**
```
Vendor: Office Supply Warehouse
```

**Test PO:**
```
Vendor: OSW Inc
```

**Historical Data (from knowledge base):**
```
Past exceptions show "Office Supply Warehouse" was resolved to "OSW Inc"
by AP clerks 8 times in last 12 months with 100% success rate.
```

**Expected AI Response:**
- recommended_action: "Auto-Resolve"
- confidence: 0.80-0.89 (lower than exact DB match)
- reasoning: Should mention "historical pattern" or "past resolutions"

**Validation:**
- ✓ AI learns from historical data (not just known alias DB)
- ✓ Confidence appropriately lower than exact match
- ✓ Workflow applies correction

---

### IT-3: Amount Mismatch - Within 5% Tolerance

**Test Invoice:**
```
Invoice Amount: $1,050.00
PO Amount: $1,000.00
Difference: 5.0%
```

**Expected AI Response:**
- recommended_action: "Auto-Resolve"
- confidence: ≥ 0.85
- exception_type: "AmountMismatch"
- correction_data.tolerance_applied: 5.0
- reasoning: Should mention "within 5% tolerance" or "likely shipping/tax"

**Validation:**
- ✓ CalculateAmountDifference tool returns 5.0%
- ✓ AI recommends auto-resolve
- ✓ Workflow proceeds without amount correction (tolerance applied)

---

### IT-4: Amount Mismatch - Exceeds 5% Tolerance

**Test Invoice:**
```
Invoice Amount: $1,200.00
PO Amount: $1,000.00
Difference: 20%
```

**Expected AI Response:**
- recommended_action: "Route-to-AP-Manager"
- confidence: Any (doesn't matter, requires human review)
- exception_type: "AmountMismatch"
- reasoning: Should mention "exceeds tolerance" or "requires investigation"

**Validation:**
- ✓ CalculateAmountDifference tool returns 20%
- ✓ AI correctly identifies out-of-tolerance
- ✓ Routed to manual review queue

---

### IT-5: Duplicate Invoice Detection

**Test Invoice:**
```
Invoice Number: INV-DUPLICATE-001
Vendor: Test Vendor Inc
```

**QuickBooks Data:**
```
Bill already exists with DocNumber "INV-DUPLICATE-001", VendorRef "Test Vendor Inc"
Processed date: 2026-01-20
QuickBooks ID: BILL-9999
```

**Expected AI Response:**
- recommended_action: "Auto-Reject"
- confidence: ≥ 0.95 (high confidence for exact match)
- exception_type: "Duplicate"
- correction_data.duplicate_date: "2026-01-20"
- reasoning: Should cite specific QuickBooks reference and date

**Validation:**
- ✓ CheckDuplicateInvoice tool returns is_duplicate: true
- ✓ AI recommends auto-reject
- ✓ Workflow sets transaction status to BusinessRuleException
- ✓ Invoice not uploaded to QuickBooks

---

### IT-6: PO Not Found - Format Variation

**Test Invoice:**
```
PO Number: 12345 (no prefix)
```

**SAP Data:**
```
PO "12345" not found
PO "PO-12345" exists (format variation)
```

**Expected AI Response:**
- recommended_action: "Auto-Resolve"
- confidence: ≥ 0.85
- exception_type: "PONotFound"
- correction_data.po_format_variation: "PO-12345"
- reasoning: Should mention "format variation detected"

**Validation:**
- ✓ LookupPOByFormat tool tries variations and finds "PO-12345"
- ✓ AI recommends auto-resolve with corrected format
- ✓ Workflow retries validation with "PO-12345"

---

### IT-7: PO Not Found - Truly Missing (Not Format Issue)

**Test Invoice:**
```
PO Number: 99999
```

**SAP Data:**
```
PO "99999" not found
Tried variations: "PO-99999", "099999" - none found
```

**Expected AI Response:**
- recommended_action: "Route-to-AP-Manager"
- confidence: Any
- exception_type: "PONotFound"
- reasoning: Should mention "PO not found in SAP" and "may require PO creation"

**Validation:**
- ✓ LookupPOByFormat tool exhausts all variations
- ✓ AI correctly identifies no PO exists
- ✓ Routed to manual review

---

### IT-8: High Amount Blocks Auto-Resolve (Guardrail)

**Test Invoice:**
```
Invoice Amount: $7,500
```

**AI Response:**
```json
{
  "recommended_action": "Auto-Resolve",
  "confidence": 0.95,
  "exception_type": "VendorAlias"
}
```

**Expected Workflow Behavior:**
- ✓ AI recommendation received
- ✓ Guardrail check fails (amount > $5,000)
- ✓ Recommendation overridden to "Route-to-AP-Manager"
- ✓ Reasoning appended: "[Auto-resolution blocked: Invoice amount exceeds $5,000 limit]"
- ✓ Routed to manual review with AI context

---

### IT-9: Unknown Exception Type

**Test Invoice:**
```
Validation Error: "Line item quantity mismatch" (unusual error)
```

**Expected AI Response:**
- recommended_action: "Route-to-AP-Manager" or "Route-to-Vendor"
- confidence: Low (<0.75)
- exception_type: "Unknown" or new type
- reasoning: Should explain lack of pattern match

**Validation:**
- ✓ AI handles gracefully (doesn't fail)
- ✓ Conservative recommendation (manual review)
- ✓ Low confidence reflects uncertainty

---

### IT-10: AI Agent Response Time (Performance)

**Test Setup:**
- 10 sequential invoice exceptions
- Measure AI response time for each

**Expected Result:**
- ✓ 95th percentile response time < 5 seconds
- ✓ Average response time < 3 seconds
- ✓ No timeouts

**Validation:**
- Review logs for response time
- All invocations complete within timeout

---

## 3. End-to-End Tests

**Purpose:** Test full invoice processing flow from download to resolution

**Prerequisites:**
- Full automation deployed to Dev
- Test email account with invoice emails
- Test vendor portal with invoices
- Test SAP and QuickBooks instances

---

### E2E-1: Happy Path with AI Auto-Resolve (Vendor Alias)

**Test Scenario:**
1. Email with invoice INV-E2E-001 arrives in AP inbox
2. Invoice vendor: "Acme Inc"
3. PO vendor: "Acme Incorporated"
4. Amount: $1,200 (matches PO)

**Expected Flow:**
1. ✓ Email trigger adds to InvoiceProcessing queue
2. ✓ DownloadInvoice.xaml downloads PDF
3. ✓ ExtractInvoiceData.xaml extracts invoice data
4. ✓ ValidateAgainstPO.xaml fails (vendor mismatch)
5. ✓ AnalyzeExceptionWithAI.xaml invoked
6. ✓ AI recommends "Auto-Resolve" with confidence 0.92
7. ✓ Guardrails pass
8. ✓ Vendor name corrected to "Acme Incorporated"
9. ✓ UploadToQuickBooks.xaml creates bill
10. ✓ Transaction status: Success
11. ✓ Invoice archived

**Validation:**
- ✓ Invoice successfully processed end-to-end
- ✓ QuickBooks bill created with correct vendor name
- ✓ Logs show AI auto-resolution
- ✓ No items in InvoiceExceptions queue

---

### E2E-2: AI Routes to Manual Review (Low Confidence)

**Test Scenario:**
1. Invoice with unusual vendor name variation
2. No historical pattern, not in alias database
3. AI cannot confidently resolve

**Expected Flow:**
1-4. Same as E2E-1 (download, extract, validate)
5. ✓ AnalyzeExceptionWithAI.xaml invoked
6. ✓ AI recommends "Route-to-AP-Manager" with confidence 0.65
7. ✓ Exception added to InvoiceExceptions queue with AI reasoning
8. ✓ Transaction status: BusinessRuleException

**Validation:**
- ✓ Item appears in InvoiceExceptions queue
- ✓ Queue item includes AI reasoning and confidence
- ✓ AP clerk can review AI's analysis
- ✓ Invoice NOT uploaded to QuickBooks

---

### E2E-3: AI Auto-Rejects Duplicate

**Test Scenario:**
1. Invoice INV-DUPLICATE-123 previously processed on Jan 20
2. Same invoice submitted again

**Expected Flow:**
1-4. Same as E2E-1 (download, extract, validate)
5. ✓ AnalyzeExceptionWithAI.xaml invoked
6. ✓ AI calls CheckDuplicateInvoice tool
7. ✓ Tool returns is_duplicate: true
8. ✓ AI recommends "Auto-Reject"
9. ✓ Transaction status: BusinessRuleException (duplicate)
10. ✓ Logged: "AI auto-rejecting invoice: Duplicate detected..."

**Validation:**
- ✓ Invoice rejected automatically
- ✓ NOT added to exception queue (clear duplicate)
- ✓ NOT uploaded to QuickBooks
- ✓ Log shows duplicate detection with original date

---

### E2E-4: AI Failure Graceful Degradation

**Test Scenario:**
1. AI Agent Builder service temporarily unavailable
2. Invoice fails validation

**Expected Flow:**
1-4. Same as E2E-1
5. ✓ AnalyzeExceptionWithAI.xaml invoked
6. ✓ AI agent call times out after 5 seconds
7. ✓ CATCH block triggered
8. ✓ Fallback recommendation: "Route-to-AP-Manager"
9. ✓ Exception added to queue (without AI reasoning)
10. ✓ Transaction status: BusinessRuleException

**Validation:**
- ✓ Automation continues despite AI failure
- ✓ Item routed to manual review (current v1.0 behavior)
- ✓ Log warning: "AI agent unavailable, routing to manual review"
- ✓ No automation downtime

---

### E2E-5: Amount Mismatch Within Tolerance

**Test Scenario:**
1. Invoice amount: $1,050
2. PO amount: $1,000
3. Difference: 5% (at tolerance limit)

**Expected Flow:**
1-4. Same as E2E-1
5. ✓ AnalyzeExceptionWithAI.xaml invoked
6. ✓ AI calls CalculateAmountDifference tool (returns 5%)
7. ✓ AI recommends "Auto-Resolve" (tolerance applied)
8. ✓ Guardrails pass
9. ✓ No amount correction needed (invoice amount used)
10. ✓ UploadToQuickBooks.xaml creates bill with invoice amount
11. ✓ Transaction status: Success

**Validation:**
- ✓ Invoice processed successfully
- ✓ QuickBooks bill shows $1,050 (invoice amount, not PO amount)
- ✓ Logs show tolerance applied

---

## 4. Shadow Mode Validation

**Purpose:** Validate AI recommendations against human decisions before enabling auto-resolution

**Duration:** 2 weeks

**Prerequisites:**
- Config.xlsx: AIShadowMode = true
- Automation deployed to production-like environment
- AP clerks available to review and resolve exceptions

---

### Shadow Mode Test Plan

**Week 1:**

**Day 1-2: Baseline Collection**
1. Run automation on 50-100 invoices
2. All exceptions route to manual review (normal v1.0 behavior)
3. **Additionally:** AI analyzes each exception and logs recommendation
4. Capture for each exception:
   - Invoice details
   - Exception type
   - AI recommendation, confidence, reasoning
   - AP clerk decision (what they actually did)
   - Outcome (was clerk's decision correct?)

**Day 3-5: Analysis**
1. Compare AI recommendations vs. AP clerk decisions:
   ```
   Agreement Rate = (AI matches clerk) / (total exceptions) * 100
   Target: ≥ 85% agreement
   ```

2. Review disagreements:
   - False positives: AI said Auto-Resolve, clerk said needs review
   - False negatives: AI said Route-to-Manual, clerk said could auto-resolve
   - For each disagreement: Was AI or clerk correct?

3. Calculate calibrated accuracy:
   ```
   For confidence bucket 0.90-1.0:
     Accuracy = (Correct auto-resolve recommendations) / (Total in bucket)
     Target: ≥ 90% accuracy

   For confidence bucket 0.80-0.89:
     Accuracy = ?
     Target: 80-89% accuracy (calibrated)
   ```

**Week 2:**

**Day 6-7: Agent Tuning (if needed)**
- If accuracy < 90%, tune agent:
  - Review false positives → increase confidence threshold?
  - Review false negatives → enrich knowledge base?
  - Update vendor alias database with new patterns

**Day 8-10: Validation Run**
1. Re-run with tuned agent (if tuning was done)
2. Process 50-100 more invoices
3. Validate accuracy improved

**Day 11-14: Stakeholder Review**
1. Present findings to Jane (AP Manager) and Sarah (AP clerk)
2. Review edge cases and ambiguous scenarios
3. Obtain sign-off for pilot rollout

---

### Shadow Mode Success Criteria

| Metric | Target | Measured | Status |
|--------|--------|----------|--------|
| **AI vs. Clerk Agreement** | ≥ 85% | ___ | ___ |
| **Auto-Resolve Accuracy (High Confidence)** | ≥ 90% | ___ | ___ |
| **Confidence Calibration (0.90-1.0 bucket)** | 90-100% | ___ | ___ |
| **Confidence Calibration (0.80-0.89 bucket)** | 80-89% | ___ | ___ |
| **False Positive Rate** | < 5% | ___ | ___ |
| **AI Response Time (p95)** | < 5s | ___ | ___ |
| **Stakeholder Confidence** | Approved | ___ | ___ |

**Decision Point:**
- If all criteria met → Proceed to Pilot (Phase 4)
- If criteria not met → Additional tuning or scope adjustment

---

## 5. Performance Tests

**Purpose:** Validate AI agent can handle production volume without degradation

---

### PERF-1: Volume Test (100 Invoices)

**Test Setup:**
1. Queue 100 invoices (mix of valid and exceptions)
2. Run automation continuously

**Expected Result:**
- ✓ All invoices processed successfully
- ✓ AI response time < 5s (p95)
- ✓ No AI timeouts
- ✓ No memory leaks or resource exhaustion

**Metrics to Capture:**
- Total processing time
- AI invocation count
- Average/p95/max AI response time
- Exception count (validation failures)
- Auto-resolve count
- Route-to-manual count

---

### PERF-2: Concurrent Processing (10 Parallel Robots)

**Test Setup:**
1. 10 unattended robots processing InvoiceProcessing queue
2. 200 invoices total (mix of valid and exceptions)

**Expected Result:**
- ✓ All robots can invoke AI agent concurrently
- ✓ No AI agent throttling or rate limiting
- ✓ AI response time remains < 5s (p95)
- ✓ No transaction conflicts

---

### PERF-3: AI Agent Load (Stress Test)

**Test Setup:**
1. 500 invoices queued (many with exceptions)
2. Run automation at max throughput

**Expected Result:**
- ✓ AI agent handles spike in requests
- ✓ Response time degradation graceful (if any)
- ✓ Graceful degradation to manual review if AI overloaded
- ✓ No automation failures due to AI issues

---

## Appendix A: Test Data Setup

### Test Invoice Templates

**Template 1: Vendor Alias Exception**
```json
{
  "invoice_number": "INV-TEST-ALIAS-001",
  "invoice_vendor": "Acme Inc",
  "invoice_amount": 1200.00,
  "po_number": "PO-5678",
  "po_vendor": "Acme Incorporated",
  "po_amount": 1200.00,
  "expected_exception": "Vendor mismatch",
  "expected_ai_recommendation": "Auto-Resolve"
}
```

**Template 2: Amount Mismatch (Within Tolerance)**
```json
{
  "invoice_amount": 1050.00,
  "po_amount": 1000.00,
  "difference_percent": 5.0,
  "expected_ai_recommendation": "Auto-Resolve"
}
```

**Template 3: Duplicate Invoice**
```json
{
  "invoice_number": "INV-DUPLICATE-001",
  "quickbooks_existing_bill": "BILL-9999",
  "existing_bill_date": "2026-01-20",
  "expected_ai_recommendation": "Auto-Reject"
}
```

---

## Appendix B: Test Execution Checklist

### Pre-Test Setup
- [ ] Workflows deployed to Dev/Test environment
- [ ] InvoiceExceptionAnalyzer agent deployed
- [ ] Test data loaded (vendor aliases, historical exceptions)
- [ ] Config.xlsx updated with test settings
- [ ] Test QuickBooks instance configured
- [ ] Test SAP instance accessible

### Unit Test Execution
- [ ] UT-1: High confidence auto-resolve - PASS/FAIL
- [ ] UT-2: Amount guardrail - PASS/FAIL
- [ ] UT-3: Low confidence - PASS/FAIL
- [ ] UT-4: Timeout graceful degradation - PASS/FAIL
- [ ] UT-5: Invalid JSON - PASS/FAIL
- [ ] UT-6: Auto-reject duplicate - PASS/FAIL

### Integration Test Execution
- [ ] IT-1 through IT-10: All integration tests - PASS/FAIL

### E2E Test Execution
- [ ] E2E-1 through E2E-5: All end-to-end tests - PASS/FAIL

### Shadow Mode Validation
- [ ] Week 1 baseline collection - COMPLETE
- [ ] Analysis and tuning - COMPLETE
- [ ] Week 2 validation - COMPLETE
- [ ] Stakeholder sign-off - APPROVED

### Performance Tests
- [ ] PERF-1: Volume test - PASS/FAIL
- [ ] PERF-2: Concurrent processing - PASS/FAIL
- [ ] PERF-3: Stress test - PASS/FAIL

### Sign-Off
- [ ] All critical tests passing (UT, IT, E2E)
- [ ] Shadow mode metrics meet targets
- [ ] Performance acceptable for production volume
- [ ] Approved by: _____________ Date: _______

---

**END OF TEST SCENARIOS**
