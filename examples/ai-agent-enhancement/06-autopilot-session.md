# Autopilot Session: Building AnalyzeExceptionWithAI.xaml

**Engineer:** Mike Chen
**Date:** February 3, 2026
**Tool:** UiPath Studio 2024.10 + Autopilot
**Reference:** Plan.md Section 2

---

## Session Overview

This document shows how to use UiPath Autopilot with the detailed prompts from Plan.md to generate the `AnalyzeExceptionWithAI.xaml` workflow.

**Key Learning:** The more detailed and specific the prompt, the better Autopilot's output. Plan.md provides prescriptive prompts with:
- Exact activity names
- Argument and variable names
- Step-by-step logic
- Error handling patterns

---

## Step 1: Open UiPath Studio and Create Workflow

1. Open UiPath Studio
2. Open the Invoice Approval project
3. Right-click on project → Add → Workflow
4. Name: `AnalyzeExceptionWithAI.xaml`
5. Click Create

---

## Step 2: Open Autopilot Panel

1. In Studio, click **Autopilot** button (top right, AI sparkle icon)
2. Autopilot panel opens on the right side
3. Ensure you're in the AnalyzeExceptionWithAI.xaml workflow

---

## Step 3: Paste Autopilot Prompt

1. Copy the complete Autopilot prompt from **Plan.md Section 2**

   **Tip:** The prompt starts with:
   ```
   WORKFLOW NAME: AnalyzeExceptionWithAI.xaml

   PURPOSE: Invokes an AI agent to analyze invoice validation exceptions...
   ```

2. Paste the **entire prompt** into Autopilot chat

3. Click **Generate** or press Enter

---

## Step 4: Review Generated Workflow

**Autopilot will generate:**

### Arguments (Input/Output)

Autopilot should create these arguments automatically:

**Input:**
- `in_InvoiceData` (DataTable)
- `in_ValidationErrors` (String)
- `in_POData` (DataRow)
- `in_InvoiceFilePath` (String)
- `in_Config` (Dictionary<String,String>)

**Output:**
- `out_RecommendedAction` (String)
- `out_Reasoning` (String)
- `out_Confidence` (Double)
- `out_ExceptionType` (String)
- `out_CorrectionData` (Newtonsoft.Json.Linq.JObject)

**✓ Verify:** Check Arguments panel - all arguments present with correct types

---

### Variables

Autopilot should create these variables in Main Sequence scope:

- `str_AgentSkillName` (String)
- `str_AgentPrompt` (String)
- `obj_AgentContext` (Newtonsoft.Json.Linq.JObject)
- `str_AgentResponseJSON` (String)
- `obj_AgentResponse` (Newtonsoft.Json.Linq.JObject)
- `dbl_ConfidenceThreshold` (Double)
- `dbl_AutoResolveThreshold` (Double)
- `dbl_MaxAutoResolveAmount` (Double)
- `dbl_InvoiceAmount` (Double)
- `bool_GuardrailsPassed` (Boolean)

**✓ Verify:** Check Variables panel - all variables present with correct types and scopes

---

### Workflow Structure

Autopilot should generate this structure:

```
Sequence: Main Sequence
│
├── Log Message: "AI Agent: Analyzing exception for invoice..."
│
├── Assign: Load configuration thresholds
│   (5 assignments: dbl_ConfidenceThreshold, dbl_AutoResolveThreshold, etc.)
│
├── Try-Catch
│   │
│   ├── TRY BLOCK:
│   │   │
│   │   ├── Assign: Build AI agent context (JObject)
│   │   │
│   │   ├── Assign: Build AI agent prompt
│   │   │
│   │   ├── Invoke AI Agent (UiPath.AI.Activities)
│   │   │   - Timeout: 5000ms
│   │   │
│   │   ├── Log Message: "AI Agent response received"
│   │   │
│   │   ├── Deserialize JSON
│   │   │
│   │   ├── Assign: Extract AI response fields
│   │   │
│   │   ├── Log Message: "AI Recommendation: {action}..."
│   │   │
│   │   └── If: Check guardrails for auto-resolution
│   │       │
│   │       ├── THEN: Guardrails passed
│   │       │   └── Assign: bool_GuardrailsPassed = True
│   │       │
│   │       └── ELSE: Guardrails failed
│   │           └── If: Check amount guardrail
│   │               └── If: Check confidence threshold
│   │
│   ├── CATCH BLOCK (System.Exception):
│   │   │
│   │   ├── Log Message: "AI Agent failed..."
│   │   │
│   │   └── Assign: Fallback to manual review
│   │
│   └── FINALLY BLOCK:
│       └── Log Message: "AI Agent workflow completed"
```

**✓ Verify:** Workflow Designer shows this structure

---

## Step 5: Fine-Tune Generated Workflow

**Even with detailed prompts, Autopilot may need minor adjustments:**

### Common Adjustments:

#### 1. Invoke AI Agent Activity

Autopilot might generate a placeholder. **Replace with actual activity:**

1. Delete placeholder (if any)
2. From Activities Panel, search: "Invoke AI Agent"
3. Drag **Invoke AI Agent** (from UiPath.AI.Activities package) into workflow
4. Configure properties:
   - **Agent Skill Name:** `str_AgentSkillName`
   - **Input Prompt:** `str_AgentPrompt`
   - **Additional Context (Optional):** `obj_AgentContext.ToString`
   - **Output:** `str_AgentResponseJSON`
   - **Timeout:** `5000` (milliseconds)

**Screenshot Reference:**

```
[Invoke AI Agent Activity Properties]
───────────────────────────────────
Agent Skill Name: str_AgentSkillName
Input Prompt: str_AgentPrompt
Additional Context: obj_AgentContext.ToString
Output: str_AgentResponseJSON
Timeout (ms): 5000
───────────────────────────────────
```

#### 2. JObject Initialization

Autopilot might not correctly generate the JObject context. **Verify or fix:**

```vb
Assign: obj_AgentContext
Value:
New JObject From {
    New JProperty("invoice_number", in_InvoiceData.Rows(0)("InvoiceNumber").ToString),
    New JProperty("invoice_vendor", in_InvoiceData.Rows(0)("VendorName").ToString),
    New JProperty("invoice_amount", dbl_InvoiceAmount),
    New JProperty("invoice_date", in_InvoiceData.Rows(0)("InvoiceDate").ToString),
    New JProperty("po_number", in_InvoiceData.Rows(0)("PONumber").ToString),
    New JProperty("po_vendor", If(in_POData IsNot Nothing, in_POData("Vendor").ToString, "")),
    New JProperty("po_amount", If(in_POData IsNot Nothing, Convert.ToDouble(in_POData("Total")), 0.0)),
    New JProperty("validation_error", in_ValidationErrors),
    New JProperty("invoice_pdf_path", in_InvoiceFilePath)
}
```

**Note:** Ensure `Newtonsoft.Json.Linq` namespace is imported

#### 3. String Formatting in Log Messages

Autopilot might use simple concatenation. **Consider using String.Format:**

```vb
Before (Autopilot default):
"AI Recommendation: " + out_RecommendedAction + " (confidence: " + out_Confidence.ToString + ")"

After (cleaner):
String.Format("AI Recommendation: {0} (confidence: {1:P0})", out_RecommendedAction, out_Confidence)
```

#### 4. Null Safety in PO Data

Autopilot might not handle `in_POData = Nothing` case. **Verify If conditions:**

```vb
If(in_POData IsNot Nothing, in_POData("Vendor").ToString, "")
```

---

## Step 6: Test the Generated Workflow

### Unit Test with Mock Data

1. Create test harness workflow: `Test_AnalyzeExceptionWithAI.xaml`

2. Build mock input data:
```vb
Mock Invoice DataTable:
dt_TestInvoice.Rows.Add("INV-001", "Acme Inc", 1200.00, "PO-5678", ...)

Mock Validation Error:
str_TestError = "Vendor mismatch: Acme Inc != Acme Incorporated"

Mock PO DataRow:
dr_TestPO("Vendor") = "Acme Incorporated"
dr_TestPO("Total") = 1200.00

Mock Config:
dict_Config("AIAgentSkillName") = "InvoiceExceptionAnalyzer"
dict_Config("AIConfidenceThreshold") = "0.75"
dict_Config("AIAutoResolveThreshold") = "0.85"
dict_Config("AIMaxAutoResolveAmount") = "5000"
```

3. **Option A:** Mock AI Agent Response (for unit testing)
   - Temporarily replace "Invoke AI Agent" with mock JSON response:
   ```vb
   str_AgentResponseJSON = "{""recommended_action"":""Auto-Resolve"",""reasoning"":""Vendor alias detected"",""confidence"":0.92,""exception_type"":""VendorAlias"",""correction_data"":{""corrected_vendor_name"":""Acme Incorporated""}}"
   ```

4. **Option B:** Use Real AI Agent (for integration testing)
   - Ensure InvoiceExceptionAnalyzer agent is deployed
   - Use real API key from Orchestrator asset

5. Invoke workflow:
```vb
Invoke Workflow: AnalyzeExceptionWithAI.xaml
Inputs: dt_TestInvoice, str_TestError, dr_TestPO, "C:\test\invoice.pdf", dict_Config
Outputs: str_Recommendation, str_Reasoning, dbl_Confidence, str_ExceptionType, obj_Correction
```

6. **Verify outputs:**
   - `str_Recommendation` = "Auto-Resolve"
   - `dbl_Confidence` = 0.92
   - `obj_Correction("corrected_vendor_name")` = "Acme Incorporated"

7. **Check logs:**
   - "AI Agent: Analyzing exception for invoice INV-001"
   - "AI Recommendation: Auto-Resolve (confidence: 92%)"
   - "Guardrails passed, proceeding with auto-resolution"

---

## Step 7: Common Issues & Troubleshooting

### Issue 1: "Invoke AI Agent" activity not found

**Solution:**
1. Go to **Manage Packages**
2. Search for: `UiPath.AI.Activities`
3. Install version **2.0.0 or later**
4. Close and reopen workflow
5. Activity should now appear in Activities panel

---

### Issue 2: JObject type not recognized

**Solution:**
1. Add **Imports** at workflow level:
   - `Newtonsoft.Json.Linq`
2. Or use fully qualified type:
   - `Newtonsoft.Json.Linq.JObject`

---

### Issue 3: AI Agent timeout

**Symptoms:** Workflow waits 5 seconds, then jumps to CATCH block

**Solutions:**
1. **Check Agent Deployment:**
   - Log into Agent Builder console
   - Verify "InvoiceExceptionAnalyzer" is deployed and running

2. **Check API Key:**
   - Verify Orchestrator asset `AIAgentAPIKey` exists
   - Value should be valid API key from Agent Builder

3. **Check Network:**
   - Ensure robot can reach Agent Builder endpoint
   - Check firewall rules

4. **Increase Timeout (if needed):**
   - Update Config.xlsx: `AIAgentTimeout = 10` (from 5)
   - Only if agent legitimately needs more time

---

### Issue 4: Invalid JSON parsing

**Symptoms:** Deserialize JSON throws exception

**Debug Steps:**
1. Add **Log Message** before Deserialize JSON:
   ```vb
   "Raw AI Response: " + str_AgentResponseJSON
   ```
2. Check Orchestrator logs - is JSON valid?
3. Common issues:
   - Missing required fields (recommended_action, confidence, etc.)
   - Malformed JSON (extra commas, missing quotes)
4. **Solution:** Improve AI agent prompt to ensure valid JSON output

---

## Step 8: Iterate with Autopilot (If Needed)

If generated workflow needs significant changes:

1. **Refine the prompt:**
   - Add more specific details about the issue
   - Example: "Use Try-Catch with System.Exception type, not generic exception"

2. **Regenerate specific sections:**
   - Select activity/sequence in Designer
   - Right-click → "Ask Autopilot to modify"
   - Provide refined instruction

3. **Learn from corrections:**
   - If you manually fix something, note the pattern
   - Update prompts in Plan.md for future use

---

## Step 9: Save and Publish

1. **Save workflow:** Ctrl+S

2. **Validate project:**
   - Click **Validate** (Ctrl+F6)
   - Fix any errors (should be none if Autopilot worked well)

3. **Publish to Orchestrator:**
   - Click **Publish**
   - Target: Dev environment
   - Version: 2.0.1-dev

4. **Test in Orchestrator:**
   - Add test invoice to InvoiceProcessing queue
   - Trigger job
   - Monitor logs for AI agent invocation

---

## Autopilot Usage Summary

### What Worked Well:
✓ Generated complete workflow structure (Try-Catch, If conditions, Assigns)
✓ Created all arguments and variables with correct types
✓ Generated log messages with proper formatting
✓ Implemented graceful degradation pattern

### What Needed Manual Adjustment:
⚠ Invoke AI Agent activity (replaced placeholder with actual activity)
⚠ JObject initialization syntax (minor VB.NET corrections)
⚠ Import namespaces (Newtonsoft.Json.Linq)

### Time Savings:
- **Without Autopilot:** ~2-3 hours to build this workflow manually
- **With Autopilot:** ~30 minutes (generate + review + adjust + test)
- **Savings:** ~70-85% time reduction

### Key Takeaway:
**Detailed, prescriptive prompts are essential.** The Spec Agent's Plan.md provided:
- Exact activity names
- Complete logic flow
- Argument/variable names
- Error handling patterns

This level of detail enabled Autopilot to generate 90%+ correct code on first try.

---

## Next Steps

1. ✓ AnalyzeExceptionWithAI.xaml built and tested
2. → **Next:** Modify Process.xaml using Autopilot prompt from Plan.md Section 3
3. → **Next:** Configure InvoiceExceptionAnalyzer agent in Agent Builder (Plan.md Section 4)
4. → **Next:** Run integration tests with real AI agent

---

## Reference

- **Detailed Autopilot Prompt:** [Plan.md Section 2](./04-plan.md#2-workflow-analyzeexceptionwithaixaml)
- **Autopilot Guide:** [studio/AUTOPILOT_GUIDE.md](../../studio/AUTOPILOT_GUIDE.md)
- **Test Scenarios:** [05-test-scenarios.md](./05-test-scenarios.md)
