# Implementation Plan: AI-Powered Exception Analysis

**Project:** Invoice Approval Automation v2.0
**Story ID:** IAUTO-247
**Date:** February 2, 2026
**Author:** Spec Agent
**Engineer:** Mike Chen

---

## Table of Contents
1. [Architecture Overview](#1-architecture-overview)
2. [Workflow: AnalyzeExceptionWithAI.xaml](#2-workflow-analyzeexceptionwithaixaml)
3. [Workflow Modification: Process.xaml](#3-workflow-modification-processxaml)
4. [AI Agent Configuration](#4-ai-agent-configuration)
5. [Configuration Updates](#5-configuration-updates)
6. [Deployment Plan](#6-deployment-plan)

---

## 1. Architecture Overview

### Integration Point

The AI agent workflow integrates into the existing REFramework at the exception handling point:

```
Process.xaml (Current flow)
├── DownloadInvoice.xaml
├── ExtractInvoiceData.xaml
├── ValidateAgainstPO.xaml
│   └── IF out_IsValid = False:
│       ├── AnalyzeExceptionWithAI.xaml  ← NEW WORKFLOW
│       │   ├── IF AI recommends "Auto-Resolve" AND guardrails pass:
│       │   │   ├── Apply correction
│       │   │   └── UploadToQuickBooks.xaml (continue happy path)
│       │   └── ELSE:
│       │       └── Add to InvoiceExceptions queue (with AI reasoning)
│       └── ELSE (validation passed):
│           ├── CheckApprovalThreshold.xaml
│           └── UploadToQuickBooks.xaml
```

### Data Flow

```
Validation Failed
     ↓
Build AI Context (invoice data + PO data + error)
     ↓
Invoke AI Agent (5s timeout)
     ↓
Parse AI Response
     ↓
Check Guardrails (amount ≤ $5K, confidence ≥ 85%)
     ↓
Apply Recommendation:
  - Auto-Resolve → Apply correction, continue to upload
  - Auto-Reject → Log as duplicate, skip
  - Route-to-Manual → Add to exception queue with AI reasoning
     ↓
Graceful Degradation (on AI failure → route to manual review)
```

---

## 2. Workflow: AnalyzeExceptionWithAI.xaml

### Autopilot Prompt

```
WORKFLOW NAME: AnalyzeExceptionWithAI.xaml

PURPOSE: Invokes an AI agent to analyze invoice validation exceptions and recommend resolution actions with confidence scores.

ARGUMENTS:
Input Arguments:
- in_InvoiceData (DataTable): Extracted invoice fields from ExtractInvoiceData workflow
- in_ValidationErrors (String): Error description from ValidateAgainstPO workflow
- in_POData (DataRow): Purchase order data from SAP (if PO was found)
- in_InvoiceFilePath (String): Full path to invoice PDF
- in_Config (Dictionary<String,String>): Configuration settings from Config.xlsx

Output Arguments:
- out_RecommendedAction (String): One of "Auto-Resolve", "Auto-Reject", "Route-to-AP-Manager", "Route-to-Vendor"
- out_Reasoning (String): Natural language explanation from AI agent
- out_Confidence (Double): Confidence score from 0.0 to 1.0
- out_ExceptionType (String): Classified exception type (VendorAlias, AmountMismatch, Duplicate, PONotFound)
- out_CorrectionData (JObject): Structured correction details (if auto-resolvable)

VARIABLES:
- str_AgentSkillName (String, Main Sequence): Name of the AI agent skill from Config
- str_AgentPrompt (String, Main Sequence): Formatted prompt for AI agent
- obj_AgentContext (JObject, Main Sequence): Structured context for AI agent
- str_AgentResponseJSON (String, Main Sequence): Raw JSON response from AI agent
- obj_AgentResponse (JObject, Main Sequence): Parsed AI agent response
- dbl_ConfidenceThreshold (Double, Main Sequence): Min confidence to accept recommendation (from Config)
- dbl_AutoResolveThreshold (Double, Main Sequence): Min confidence for auto-resolution (from Config)
- dbl_MaxAutoResolveAmount (Double, Main Sequence): Max invoice amount for auto-resolution (from Config)
- dbl_InvoiceAmount (Double, Main Sequence): Grand total from invoice
- bool_GuardrailsPassed (Boolean, Main Sequence): True if all business rules pass

ACTIVITIES AND LOGIC:

1. SEQUENCE: Main Sequence

2. LOG MESSAGE (Info): "AI Agent: Analyzing exception for invoice {InvoiceNumber}"
   - Message: "AI Agent: Analyzing exception for invoice " + in_InvoiceData.Rows(0)("InvoiceNumber").ToString

3. ASSIGN: Load configuration thresholds
   - dbl_ConfidenceThreshold = Convert.ToDouble(in_Config("AIConfidenceThreshold"))
   - dbl_AutoResolveThreshold = Convert.ToDouble(in_Config("AIAutoResolveThreshold"))
   - dbl_MaxAutoResolveAmount = Convert.ToDouble(in_Config("AIMaxAutoResolveAmount"))
   - str_AgentSkillName = in_Config("AIAgentSkillName").ToString
   - dbl_InvoiceAmount = Convert.ToDouble(in_InvoiceData.Rows(0)("GrandTotal"))

4. TRY-CATCH (for graceful degradation):

   TRY BLOCK:

   5. ASSIGN: Build AI agent context (JObject)
      - obj_AgentContext = New JObject From {
          {"invoice_number", in_InvoiceData.Rows(0)("InvoiceNumber").ToString},
          {"invoice_vendor", in_InvoiceData.Rows(0)("VendorName").ToString},
          {"invoice_amount", dbl_InvoiceAmount},
          {"invoice_date", in_InvoiceData.Rows(0)("InvoiceDate").ToString},
          {"po_number", in_InvoiceData.Rows(0)("PONumber").ToString},
          {"po_vendor", If(in_POData IsNot Nothing, in_POData("Vendor").ToString, "")},
          {"po_amount", If(in_POData IsNot Nothing, Convert.ToDouble(in_POData("Total")), 0.0)},
          {"validation_error", in_ValidationErrors},
          {"invoice_pdf_path", in_InvoiceFilePath}
        }

   6. ASSIGN: Build AI agent prompt
      - str_AgentPrompt = "Analyze this invoice validation exception and recommend resolution:

Invoice Details:
- Number: " + obj_AgentContext("invoice_number").ToString + "
- Vendor: " + obj_AgentContext("invoice_vendor").ToString + "
- Amount: $" + obj_AgentContext("invoice_amount").ToString + "
- PO Number: " + obj_AgentContext("po_number").ToString + "

Purchase Order Details (if available):
- Vendor: " + obj_AgentContext("po_vendor").ToString + "
- Amount: $" + obj_AgentContext("po_amount").ToString + "

Validation Error: " + in_ValidationErrors + "

Analyze and determine:
1. Exception type (VendorAlias, AmountMismatch, Duplicate, PONotFound)
2. Root cause
3. Recommended action (Auto-Resolve, Auto-Reject, Route-to-AP-Manager, Route-to-Vendor)
4. Specific reasoning with evidence
5. Confidence score (0.0-1.0)
6. Correction data (if auto-resolvable)

Output JSON format:
{
  \"recommended_action\": \"Auto-Resolve\",
  \"reasoning\": \"Detailed explanation...\",
  \"confidence\": 0.92,
  \"exception_type\": \"VendorAlias\",
  \"correction_data\": {
    \"corrected_vendor_name\": \"Acme Incorporated\",
    \"source\": \"SAP_PO\"
  }
}"

   7. INVOKE AI AGENT (UiPath.AI.Activities)
      - Activity: Invoke AI Agent
      - Agent Skill: str_AgentSkillName
      - Input Prompt: str_AgentPrompt
      - Additional Context: obj_AgentContext.ToString (as string)
      - Output: str_AgentResponseJSON
      - Timeout: 5000 milliseconds (5 seconds)

   8. LOG MESSAGE (Info): "AI Agent response received"
      - Message: "AI Agent response received in " + "response_time" + "ms"

   9. DESERIALIZE JSON
      - Activity: Deserialize JSON
      - JSON String: str_AgentResponseJSON
      - Output: obj_AgentResponse (JObject)

   10. ASSIGN: Extract AI response fields
       - out_RecommendedAction = obj_AgentResponse("recommended_action").ToString
       - out_Reasoning = obj_AgentResponse("reasoning").ToString
       - out_Confidence = Convert.ToDouble(obj_AgentResponse("confidence"))
       - out_ExceptionType = obj_AgentResponse("exception_type").ToString
       - out_CorrectionData = If(obj_AgentResponse("correction_data") IsNot Nothing, CType(obj_AgentResponse("correction_data"), JObject), New JObject)

   11. LOG MESSAGE (Info): "AI Recommendation: {action} (confidence: {score})"
       - Message: String.Format("AI Recommendation: {0} (confidence: {1:P0})", out_RecommendedAction, out_Confidence)

   12. IF ACTIVITY: Check guardrails for auto-resolution
       - Condition: out_RecommendedAction = "Auto-Resolve" AndAlso out_Confidence >= dbl_AutoResolveThreshold AndAlso dbl_InvoiceAmount <= dbl_MaxAutoResolveAmount

       THEN:
         13. ASSIGN: Guardrails passed
             - bool_GuardrailsPassed = True

         14. LOG MESSAGE (Info): "Guardrails passed, proceeding with auto-resolution"

       ELSE:
         15. IF ACTIVITY: Check if blocked by amount guardrail
             - Condition: dbl_InvoiceAmount > dbl_MaxAutoResolveAmount

             THEN:
               16. LOG MESSAGE (Warn): "Auto-resolution blocked: Invoice amount ${amount} exceeds limit ${limit}"
                   - Message: String.Format("Auto-resolution blocked: Invoice amount ${0} exceeds limit ${1}", dbl_InvoiceAmount, dbl_MaxAutoResolveAmount)

               17. ASSIGN: Override to manual review
                   - out_RecommendedAction = "Route-to-AP-Manager"
                   - out_Reasoning = out_Reasoning + " [Auto-resolution blocked: Invoice amount exceeds $5,000 limit]"

         18. IF ACTIVITY: Check if blocked by confidence threshold
             - Condition: out_Confidence < dbl_AutoResolveThreshold

             THEN:
               19. LOG MESSAGE (Warn): "Confidence {score} below threshold {threshold}, routing to manual review"
                   - Message: String.Format("Confidence {0:P0} below threshold {1:P0}, routing to manual review", out_Confidence, dbl_AutoResolveThreshold)

               20. ASSIGN: Override to manual review
                   - out_RecommendedAction = "Route-to-AP-Manager"

   CATCH BLOCK (Exception type: System.Exception, variable: ex):

   21. LOG MESSAGE (Warning): "AI Agent failed: {error}. Graceful degradation to manual review."
       - Message: "AI Agent failed: " + ex.Message + ". Graceful degradation to manual review."

   22. ASSIGN: Fallback to manual review
       - out_RecommendedAction = "Route-to-AP-Manager"
       - out_Reasoning = "AI agent unavailable. Manual review required. Original error: " + in_ValidationErrors
       - out_Confidence = 0.0
       - out_ExceptionType = "Unknown"
       - out_CorrectionData = New JObject

   FINALLY BLOCK:

   23. LOG MESSAGE (Info): "AI Agent workflow completed"

END WORKFLOW

ERROR HANDLING NOTES:
- All AI failures are caught and gracefully degraded to manual review
- Timeout (5s) automatically triggers CATCH block
- Invalid JSON response triggers CATCH block
- Missing required fields in AI response triggers CATCH block
- Guardrails are enforced AFTER AI response (defense in depth)

TESTING NOTES:
- Unit test with mocked AI responses (high/medium/low confidence)
- Integration test with real AI agent and known test scenarios
- Test timeout handling by simulating slow AI response
- Test invalid JSON handling
```

---

## 3. Workflow Modification: Process.xaml

### Autopilot Prompt

```
MODIFICATION: Process.xaml (existing workflow)

LOCATION: After ValidateAgainstPO.xaml completes

CURRENT LOGIC (simplified):
1. Invoke ValidateAgainstPO.xaml
2. IF out_IsValid = True:
     - Continue to CheckApprovalThreshold.xaml
   ELSE:
     - Add to InvoiceExceptions queue
     - Set transaction status to BusinessRuleException

NEW LOGIC TO ADD:

AFTER: ValidateAgainstPO.xaml returns out_IsValid = False

INSERT THIS LOGIC:

1. SEQUENCE: AI Exception Analysis

2. LOG MESSAGE (Info): "Validation failed, invoking AI agent for exception analysis"

3. INVOKE WORKFLOW: AnalyzeExceptionWithAI.xaml
   Input Arguments:
   - in_InvoiceData = dt_InvoiceData (from ExtractInvoiceData)
   - in_ValidationErrors = str_ValidationErrors (from ValidateAgainstPO)
   - in_POData = dr_POData (from ValidateAgainstPO, may be Nothing)
   - in_InvoiceFilePath = str_InvoiceFilePath (from DownloadInvoice)
   - in_Config = in_Config (passed through from Main)

   Output Arguments:
   - out_RecommendedAction = str_AIRecommendation
   - out_Reasoning = str_AIReasoning
   - out_Confidence = dbl_AIConfidence
   - out_ExceptionType = str_ExceptionType
   - out_CorrectionData = obj_CorrectionData

4. SWITCH ACTIVITY: Handle AI recommendation
   Expression: str_AIRecommendation

   CASE: "Auto-Resolve"
     5. LOG MESSAGE (Info): "AI auto-resolving exception: {reasoning}"
        - Message: "AI auto-resolving exception: " + str_AIReasoning

     6. SWITCH ACTIVITY: Apply correction based on exception type
        Expression: str_ExceptionType

        CASE: "VendorAlias"
          7. ASSIGN: Update vendor name in invoice data
             - dt_InvoiceData.Rows(0)("VendorName") = obj_CorrectionData("corrected_vendor_name").ToString

          8. LOG MESSAGE (Info): "Vendor name corrected to {name}"
             - Message: "Vendor name corrected to " + obj_CorrectionData("corrected_vendor_name").ToString

        CASE: "AmountMismatch"
          9. ASSIGN: Apply tolerance (no correction needed, just proceed)
             - Nothing to update, tolerance applied

          10. LOG MESSAGE (Info): "Amount mismatch within tolerance: {percent}%"
              - Message: "Amount mismatch within tolerance: " + obj_CorrectionData("tolerance_applied").ToString + "%"

        DEFAULT:
          11. LOG MESSAGE (Warn): "Unknown exception type for correction: {type}"

     12. INVOKE WORKFLOW: UploadToQuickBooks.xaml
         - in_InvoiceData = dt_InvoiceData
         - in_FilePath = str_InvoiceFilePath
         - out_QBReferenceNumber = str_QBReference

     13. LOG MESSAGE (Info): "AI auto-resolved invoice uploaded to QuickBooks: {reference}"
         - Message: "AI auto-resolved invoice uploaded to QuickBooks: " + str_QBReference

     14. SET TRANSACTION STATUS: Success

   CASE: "Auto-Reject"
     15. LOG MESSAGE (Info): "AI auto-rejecting invoice: {reasoning}"
         - Message: "AI auto-rejecting invoice: " + str_AIReasoning

     16. SET TRANSACTION STATUS: BusinessRuleException (duplicate invoice)

     17. LOG MESSAGE (Info): "Invoice marked as duplicate and skipped"

   CASE: "Route-to-AP-Manager" OR "Route-to-Vendor" (default)
     18. LOG MESSAGE (Info): "Routing to manual review: {reasoning}"
         - Message: "Routing to manual review: " + str_AIReasoning

     19. ADD QUEUE ITEM: InvoiceExceptions queue
         - ItemInformation = New Dictionary(Of String, Object) From {
             {"InvoiceNumber", dt_InvoiceData.Rows(0)("InvoiceNumber").ToString},
             {"ValidationError", str_ValidationErrors},
             {"AIRecommendation", str_AIRecommendation},
             {"AIReasoning", str_AIReasoning},
             {"AIConfidence", dbl_AIConfidence.ToString("P0")},
             {"ExceptionType", str_ExceptionType},
             {"InvoiceFilePath", str_InvoiceFilePath}
           }
         - Priority = Normal
         - DeferDate = Nothing

     20. LOG MESSAGE (Info): "Exception queued with AI context for manual review"

     21. SET TRANSACTION STATUS: BusinessRuleException

VARIABLES TO ADD TO PROCESS.XAML:
- str_AIRecommendation (String): Recommended action from AI
- str_AIReasoning (String): AI's reasoning
- dbl_AIConfidence (Double): Confidence score
- str_ExceptionType (String): Exception type classification
- obj_CorrectionData (JObject): Correction details
- str_QBReference (String): QuickBooks reference (for auto-resolved items)

ERROR HANDLING:
- If AnalyzeExceptionWithAI.xaml throws exception, it's already handled internally with graceful degradation
- Process.xaml receives "Route-to-AP-Manager" recommendation and continues normally
```

---

## 4. AI Agent Configuration

### Agent Builder Setup

**Platform:** UiPath Agent Builder
**Agent Name:** InvoiceExceptionAnalyzer
**Agent Type:** Conversational with Tools

### System Prompt

```
You are an expert Accounts Payable Operations Specialist with 10+ years of experience analyzing invoice validation exceptions. Your role is to determine the root cause of validation failures and recommend the most appropriate resolution action.

CONTEXT:
You receive invoice validation exceptions from an automated invoice approval system. The invoices have failed validation against SAP purchase orders due to mismatches in vendor names, amounts, or other discrepancies.

YOUR TASK:
1. Analyze the validation error and invoice/PO details
2. Classify the exception type (VendorAlias, AmountMismatch, Duplicate, PONotFound)
3. Determine if the exception can be auto-resolved or requires human review
4. Provide a clear, specific recommendation with supporting evidence
5. Rate your confidence in the recommendation (0.0-1.0 scale)

DECISION FRAMEWORK:

**VendorAlias (Vendor name variations):**
- Check if invoice vendor is a known alias of PO vendor
- Use CheckVendorAlias tool to query alias database
- If alias match found with confidence >90%, recommend Auto-Resolve
- Example: "Acme Inc" vs "Acme Incorporated" → Auto-Resolve if confirmed alias

**AmountMismatch (Invoice total differs from PO total):**
- Calculate percentage difference: |(InvoiceAmount - POAmount) / POAmount| * 100
- If difference ≤ 5%, likely shipping/tax → recommend Auto-Resolve
- If difference > 5%, investigate line items using CheckLineItems tool
- If line items match but totals differ, may be tax calculation → recommend Auto-Resolve
- Otherwise, recommend Route-to-AP-Manager for investigation

**Duplicate (Invoice already processed):**
- Use CheckDuplicateInvoice tool to query QuickBooks
- If invoice number + vendor match found, recommend Auto-Reject
- Include the original processing date in reasoning

**PONotFound (PO number not found in SAP):**
- Check if PO number format variation (e.g., "12345" vs "PO-12345")
- Use LookupPOByFormat tool to try alternative formats
- If PO found with variation, recommend Auto-Resolve
- Otherwise, recommend Route-to-AP-Manager (may need PO creation)

GUARDRAILS (enforced by RPA workflow, but you should respect them):
- Never recommend Auto-Resolve for invoices > $5,000
- Never recommend Auto-Reject without strong evidence (duplicate detection)
- Always provide specific reasoning, not generic responses
- Always cite evidence (tool results, historical patterns, percentages)

CONFIDENCE SCORING GUIDELINES:
- 0.90-1.0: Strong evidence (exact alias match, duplicate confirmed, <2% amount difference)
- 0.75-0.89: Good evidence (historical pattern match, 2-5% amount difference)
- 0.50-0.74: Weak evidence (possible pattern, but uncertain)
- 0.0-0.49: No evidence (route to manual review)

OUTPUT FORMAT:
Always respond with valid JSON:
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

IMPORTANT:
- Be conservative with auto-resolution recommendations
- When in doubt, recommend manual review (Route-to-AP-Manager)
- Always explain your reasoning with specific data points
- Use the provided tools to gather evidence before deciding
```

### Agent Tools

Define these custom tools in Agent Builder:

#### Tool 1: CheckVendorAlias

```json
{
  "name": "CheckVendorAlias",
  "description": "Check if two vendor names are known aliases of the same vendor",
  "parameters": {
    "invoice_vendor": {
      "type": "string",
      "description": "Vendor name from invoice"
    },
    "po_vendor": {
      "type": "string",
      "description": "Vendor name from purchase order"
    }
  },
  "returns": {
    "is_alias": "boolean",
    "confidence": "number",
    "canonical_name": "string",
    "source": "string (KnownAliases | Historical | Fuzzy Match)"
  }
}
```

**Implementation (pseudo-code for UiPath workflow called by agent):**
```
1. Query VendorAliases table (known aliases database)
2. If exact match found → return {is_alias: true, confidence: 0.95, source: "KnownAliases"}
3. Query historical exception resolutions (12 months data)
4. If this name pair was previously resolved as alias → return {is_alias: true, confidence: 0.85, source: "Historical"}
5. Apply fuzzy string matching (Levenshtein distance)
6. If similarity > 85% → return {is_alias: true, confidence: 0.70, source: "FuzzyMatch"}
7. Otherwise → return {is_alias: false, confidence: 0.0}
```

#### Tool 2: CheckDuplicateInvoice

```json
{
  "name": "CheckDuplicateInvoice",
  "description": "Check if invoice already exists in QuickBooks",
  "parameters": {
    "invoice_number": {
      "type": "string",
      "description": "Invoice number to check"
    },
    "vendor_name": {
      "type": "string",
      "description": "Vendor name"
    }
  },
  "returns": {
    "is_duplicate": "boolean",
    "existing_invoice_date": "string",
    "quickbooks_reference": "string"
  }
}
```

**Implementation:**
```
1. Call QuickBooks API: GET /v3/company/{companyId}/query
   SQL: "SELECT * FROM Bill WHERE DocNumber = '{invoice_number}' AND VendorRef.name = '{vendor_name}'"
2. If results found → return {is_duplicate: true, existing_invoice_date: "...", quickbooks_reference: "..."}
3. Otherwise → return {is_duplicate: false}
```

#### Tool 3: CalculateAmountDifference

```json
{
  "name": "CalculateAmountDifference",
  "description": "Calculate percentage difference between invoice and PO amounts",
  "parameters": {
    "invoice_amount": {
      "type": "number",
      "description": "Invoice grand total"
    },
    "po_amount": {
      "type": "number",
      "description": "PO total"
    }
  },
  "returns": {
    "difference_amount": "number",
    "difference_percent": "number",
    "within_tolerance": "boolean (≤5%)"
  }
}
```

**Implementation:**
```
1. diff_amount = invoice_amount - po_amount
2. diff_percent = abs(diff_amount / po_amount) * 100
3. within_tolerance = (diff_percent <= 5.0)
4. return {difference_amount: diff_amount, difference_percent: diff_percent, within_tolerance: within_tolerance}
```

#### Tool 4: LookupPOByFormat

```json
{
  "name": "LookupPOByFormat",
  "description": "Try alternative PO number formats in SAP lookup",
  "parameters": {
    "original_po_number": {
      "type": "string",
      "description": "PO number from invoice"
    }
  },
  "returns": {
    "found": "boolean",
    "corrected_po_number": "string",
    "po_details": "object"
  }
}
```

**Implementation:**
```
1. Try original format: SAP API GET /purchase-orders/{original_po_number}
2. If not found, try variations:
   - Add "PO-" prefix: "PO-12345"
   - Remove prefix if present: "12345" (if original was "PO-12345")
   - Add leading zeros: "0012345"
3. If any variation found → return {found: true, corrected_po_number: "...", po_details: {...}}
4. Otherwise → return {found: false}
```

### Knowledge Base

**Type:** Vector database (UiPath AI Center Knowledge Base)

**Data Sources:**

1. **Historical Exception Resolutions (12 months)**
   - Format: CSV/Excel with columns:
     - ExceptionDate, InvoiceNumber, InvoiceVendor, PONumber, POVendor
     - ExceptionType, ValidationError, APClerkResolution, OutcomeSuccess
   - Purpose: Learn patterns from how AP clerks resolved similar exceptions
   - Example row:
     ```
     2025-03-15, INV-9876, Acme Inc, PO-5432, Acme Incorporated,
     VendorAlias, "Vendor mismatch", "Updated to Acme Incorporated", TRUE
     ```

2. **Known Vendor Aliases (Top 20 vendors)**
   - Format: CSV with columns:
     - CanonicalVendorName, Alias1, Alias2, Alias3, ...
   - Example row:
     ```
     Acme Incorporated, Acme Inc, Acme Corp, ACME, Acme Intl
     ```

3. **Business Rules Documentation**
   - Amount tolerance: 5% for shipping/tax differences
   - Auto-approval limits: $5,000 max
   - Duplicate detection rules

**Ingestion:**
- Upload CSVs to AI Center Knowledge Base
- Chunk size: 512 tokens
- Embedding model: text-embedding-ada-002
- Update quarterly or when significant pattern changes detected

### Guardrails

**Hard Limits (cannot be overridden by agent):**

1. **Amount Limit**
   ```json
   {
     "rule": "if invoice_amount > 5000 then reject auto_resolve recommendation",
     "rationale": "High-value invoices require human approval"
   }
   ```

2. **Confidence Threshold**
   ```json
   {
     "rule": "if confidence < 0.75 then force route_to_manual_review",
     "rationale": "Low confidence decisions need human validation"
   }
   ```

3. **Rejection Requires Evidence**
   ```json
   {
     "rule": "if recommended_action = auto_reject then require duplicate_confirmation = true",
     "rationale": "Cannot reject invoices without clear duplicate evidence"
   }
   ```

**Soft Limits (agent should respect, but RPA enforces):**

1. **Conservative Bias**
   - When uncertain, prefer manual review over auto-resolution
   - Better to have humans review 10 resolvable cases than auto-resolve 1 incorrect case

2. **Explanation Quality**
   - Reasoning must be specific (cite tool results, percentages, dates)
   - Generic responses like "seems reasonable" are not acceptable

---

## 5. Configuration Updates

### Config.xlsx Updates

**Settings Sheet - Add these rows:**

| Name | Value | Description |
|------|-------|-------------|
| AIAgentSkillName | InvoiceExceptionAnalyzer | UiPath Agent Builder agent skill name |
| AIAgentTimeout | 5 | Max wait time for AI agent response (seconds) |
| AIConfidenceThreshold | 0.75 | Min confidence score to accept AI recommendation |
| AIAutoResolveThreshold | 0.85 | Min confidence score for auto-resolution without human review |
| AIMaxAutoResolveAmount | 5000 | Max invoice amount for auto-resolution ($) |
| AIAmountTolerancePercent | 5 | Max % difference for amount mismatch auto-resolution |
| AIShadowMode | false | If true, AI logs recommendations but doesn't auto-resolve |

**Assets Sheet - Add this row:**

| Name | Type | Description |
|------|------|-------------|
| AIAgentAPIKey | Text | API key for invoking UiPath Agent Builder agent |

### Orchestrator Assets

Create these assets in Orchestrator:

| Asset Name | Type | Folder | Description |
|------------|------|--------|-------------|
| AIAgentAPIKey | Text | /Prod/InvoiceApproval | API key for Agent Builder (from Agent Builder console) |

---

## 6. Deployment Plan

### Phase 1: Agent Builder Setup (Day 1)

**Step 1.1: Create Agent in Agent Builder**
1. Log into UiPath Agent Builder console
2. Create new agent: "InvoiceExceptionAnalyzer"
3. Copy system prompt from Section 4 → paste into Agent Builder
4. Set model: GPT-4 (or latest available)

**Step 1.2: Configure Tools**
1. Add tool: CheckVendorAlias
   - Create UiPath workflow (queries vendor alias database)
   - Register as agent tool
2. Add tool: CheckDuplicateInvoice
   - Use QuickBooks API integration
   - Register as agent tool
3. Add tool: CalculateAmountDifference
   - Simple calculation (can be inline code)
   - Register as agent tool
4. Add tool: LookupPOByFormat
   - Use SAP API integration
   - Register as agent tool

**Step 1.3: Upload Knowledge Base**
1. Prepare CSV files:
   - Historical exceptions (12 months from Sarah)
   - Known vendor aliases (top 20 from AP team)
2. Upload to AI Center Knowledge Base
3. Link knowledge base to agent

**Step 1.4: Set Guardrails**
1. Configure amount limit: $5,000
2. Configure confidence threshold: 0.75 min, 0.85 for auto-resolve
3. Test guardrails with sample inputs

**Step 1.5: Test Agent**
1. Test with known scenarios:
   - Vendor alias: "Acme Inc" → should recommend Auto-Resolve
   - Amount mismatch 3%: should recommend Auto-Resolve
   - Amount mismatch 10%: should recommend Route-to-AP-Manager
   - Duplicate invoice: should recommend Auto-Reject
2. Verify confidence scores are calibrated
3. Verify reasoning is specific and evidence-based

**Step 1.6: Deploy Agent Skill**
1. Publish agent to Orchestrator
2. Note the agent skill name: "InvoiceExceptionAnalyzer"
3. Generate API key for invocation
4. Save API key to Orchestrator asset: AIAgentAPIKey

---

### Phase 2: Workflow Development (Days 2-3)

**Step 2.1: Create AnalyzeExceptionWithAI.xaml**
1. Open UiPath Studio
2. Create new workflow file: AnalyzeExceptionWithAI.xaml
3. Open UiPath Autopilot
4. **Paste the Autopilot prompt from Section 2** (complete prompt)
5. Review generated workflow:
   - Verify all activities present (Invoke AI Agent, Try-Catch, Assigns, Logs)
   - Verify arguments and variables match spec
   - Verify timeout = 5000ms
6. Test with mock data:
   - Create test harness
   - Mock AI responses (high/medium/low confidence)
   - Verify graceful degradation on timeout

**Step 2.2: Modify Process.xaml**
1. Open Process.xaml
2. Locate the section after ValidateAgainstPO.xaml
3. Open UiPath Autopilot
4. **Paste the Autopilot prompt from Section 3** (modification prompt)
5. Review generated changes:
   - Verify AI workflow invoked correctly
   - Verify Switch activity handles all recommendation types
   - Verify correction logic for vendor alias and amount tolerance
6. Test integration:
   - Run with test invoice that fails validation
   - Verify AI agent invoked
   - Verify recommendation applied correctly

**Step 2.3: Update Config.xlsx**
1. Add new settings rows (from Section 5)
2. Set initial values:
   - AIAgentSkillName = "InvoiceExceptionAnalyzer"
   - AIAgentTimeout = 5
   - AIConfidenceThreshold = 0.75
   - AIAutoResolveThreshold = 0.85
   - AIMaxAutoResolveAmount = 5000
   - AIAmountTolerancePercent = 5
   - **AIShadowMode = true** (start in shadow mode)

---

### Phase 3: Shadow Mode Testing (Week 1-2)

**Purpose:** Validate AI recommendations without affecting production

**Step 3.1: Deploy to Dev Environment**
1. Publish workflows to Orchestrator (Dev folder)
2. Update Config.xlsx: AIShadowMode = true
3. Deploy InvoiceExceptionAnalyzer agent skill (Dev)

**Step 3.2: Run Shadow Mode**
1. Process 50-100 invoices in Dev
2. AI analyzes exceptions and logs recommendations
3. **All exceptions still routed to manual review** (shadow mode)
4. Capture logs:
   - Invoice number, exception type
   - AI recommendation, confidence, reasoning
   - Actual AP clerk decision

**Step 3.3: Validate AI Accuracy**
1. Compare AI recommendations vs. AP clerk decisions
2. Calculate accuracy: (AI correct / total exceptions) * 100
3. **Target: ≥90% accuracy**
4. Review false positives/negatives with Sarah (AP SME)
5. Tune agent if needed:
   - Adjust confidence thresholds
   - Update vendor alias database
   - Refine agent prompt

**Step 3.4: Calibrate Confidence Scores**
1. Bucket recommendations by confidence:
   - 0.90-1.0: Should be 90-100% accurate
   - 0.80-0.89: Should be 80-89% accurate
   - 0.75-0.79: Should be 75-79% accurate
2. If confidence not calibrated, retrain or adjust thresholds

---

### Phase 4: Pilot Rollout (Week 3-4)

**Purpose:** Enable auto-resolution for limited vendor set

**Step 4.1: Select Pilot Vendors**
1. Choose top 5 vendors by volume
2. Ensure good historical data for these vendors
3. Examples: Acme Inc, Office Depot, Staples, FedEx, AT&T

**Step 4.2: Update Configuration**
1. Update Config.xlsx:
   - **AIShadowMode = false** (enable live mode)
   - Consider adding "AIEnabledVendors" filter (optional)
2. Publish to Test environment

**Step 4.3: Monitor Pilot**
1. Run for 2 weeks
2. Daily monitoring:
   - Auto-resolve count
   - Auto-resolve accuracy (validate against human review)
   - False positive count (auto-resolved items that should have been reviewed)
   - Exception queue volume (should decrease)
3. Weekly review with Jane (AP Manager)

**Step 4.4: Validate Success Metrics**
| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Manual review rate | ≤20% | ___ | ___ |
| Auto-resolve accuracy | ≥90% | ___ | ___ |
| AI response time (p95) | <5s | ___ | ___ |
| False positive rate | <5% | ___ | ___ |

---

### Phase 5: Full Rollout (Week 5+)

**Step 5.1: Expand to All Vendors**
1. If pilot metrics meet targets → proceed
2. Remove vendor filter (if implemented)
3. Deploy to Prod environment
4. Monitor for 1 week intensively

**Step 5.2: Continuous Improvement**
1. Weekly performance review:
   - Review auto-resolved items
   - Review false positives/negatives
   - Update vendor alias database
2. Monthly agent retraining:
   - Add new historical data (last 30 days)
   - Retrain knowledge base
   - Republish agent skill

**Step 5.3: Success Review**
1. After 30 days, measure final metrics
2. Present results to Jane for Q2 business review
3. Document lessons learned

---

## 7. Testing Strategy

See TestScenarios.md for detailed test cases.

**Summary:**
- **Unit Tests:** AnalyzeExceptionWithAI.xaml with mocked AI responses
- **Integration Tests:** Real AI agent with known test scenarios
- **E2E Tests:** Full invoice flow from download to resolution
- **Shadow Mode:** 2-week validation in production-like environment

---

## Appendix A: Troubleshooting Guide

### Issue: AI Agent Timeout
**Symptoms:** Logs show "AI Agent failed: timeout"
**Resolution:**
1. Check Agent Builder console: is agent deployed and running?
2. Check network connectivity to Agent Builder endpoint
3. Increase timeout in Config.xlsx (AIAgentTimeout) if needed
4. Check agent logs for performance issues

### Issue: Low Confidence Scores
**Symptoms:** All AI recommendations have confidence <0.75
**Resolution:**
1. Check knowledge base: is historical data uploaded?
2. Verify vendor alias database is populated
3. Review agent prompt: ensure decision framework is clear
4. Retrain knowledge base with more recent data

### Issue: False Positives (Incorrect Auto-Resolution)
**Symptoms:** Auto-resolved invoices require reversal
**Resolution:**
1. Increase AIAutoResolveThreshold (e.g., 0.85 → 0.90)
2. Lower AIMaxAutoResolveAmount if high-value errors
3. Review AI reasoning in logs: is evidence sufficient?
4. Add specific guardrails for problematic pattern

---

## Appendix B: Rollback Plan

If AI agent causes issues in production:

1. **Immediate Rollback (< 5 minutes):**
   - Update Config.xlsx: AIShadowMode = true
   - All exceptions route to manual review (current behavior)
   - No workflow redeployment needed

2. **Full Rollback (if needed):**
   - Unpublish v2.0 workflows
   - Republish v1.0 workflows (without AI agent)
   - Remove AIAgentAPIKey from Orchestrator

3. **Investigation:**
   - Review logs for root cause
   - Fix issue in Dev environment
   - Re-test before redeployment

---

**END OF IMPLEMENTATION PLAN**

**Next Steps:**
1. ✓ Requirements approved
2. ✓ Plan created
3. → **Engineer:** Build workflows using Autopilot prompts
4. → **Engineer:** Configure AI agent in Agent Builder
5. → **Engineer:** Test and deploy per rollout plan
6. → **TDD Agent:** Update TDD.md after implementation
