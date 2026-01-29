# Using Autopilot with Plan.md

Guide for engineers on using UiPath Autopilot to implement workflows from the Spec Agent's Plan.md.

## Overview

The Spec Agent generates Plan.md with detailed workflow specifications. These specs are designed to be **Autopilot-ready**—you can paste them directly into Autopilot to generate workflows.

## Autopilot Capabilities

| Feature | Use Case |
|---------|----------|
| **Text-to-Workflow** | Generate XAML sequences from natural language |
| **Text-to-Code** | Generate coded automations |
| **Expression Generation** | Create complex expressions from descriptions |
| **API Workflow Generation** | Build workflows from API specifications |
| **Activity Summarization** | Explain what existing activities do |

> **Tip:** Autopilot is available in Studio Desktop, Studio Web, and via Orchestrator. This guide focuses on Studio Desktop usage with Plan.md.

## Workflow: Plan.md → Autopilot → XAML

### Step 1: Open Plan.md

Find the workflow you want to implement. Each workflow spec looks like:

```markdown
### ExtractInvoiceData.xaml

**Purpose**: Extract data from invoice PDF using Document Understanding

**Arguments**:
| Name | Direction | Type | Description |
|------|-----------|------|-------------|
| in_PDFPath | In | String | Path to invoice PDF |
| in_Config | In | Dictionary<String,Object> | Configuration |
| out_InvoiceData | Out | DataRow | Extracted data |

**Activities**:
1. Load Document (Document Understanding)
2. Digitize Document
3. Extract Data using ML model
4. Build output DataRow
5. Return results

**Error Handling**:
- Try-Catch around extraction
- Log errors with PDF path
- Return empty DataRow on failure
```

### Step 2: Create Autopilot Prompt

Convert the spec into a natural language prompt:

```
Create a workflow called ExtractInvoiceData.xaml using Modern Design.

Arguments:
- in_PDFPath (String, In): Path to invoice PDF file
- in_Config (Dictionary<String,Object>, In): Configuration settings  
- out_InvoiceData (DataRow, Out): Extracted invoice data

The workflow should:
1. Load the PDF document using Document Understanding
2. Digitize the document
3. Extract invoice fields (InvoiceNumber, Date, VendorName, Total) using the ML model
4. Build a DataRow with the extracted data
5. Return the DataRow as out_InvoiceData

Include Try-Catch error handling. Log any errors with the PDF path. 
If extraction fails, return an empty DataRow.

Use Log Message activities at Info level for key steps.
```

### Step 3: Generate in Autopilot

1. Open UiPath Studio
2. Open Autopilot chat panel (click the Autopilot icon in the ribbon)
3. Paste your prompt
4. Review the generated workflow preview
5. Click **Insert** to add to your project, or refine with follow-up prompts

### Step 4: Review and Refine

Autopilot generates ~70% of workflows correctly on first attempt (varies based on prompt quality, workflow complexity, and UiPath version). Check:

- [ ] Arguments match spec (names, types, directions)
- [ ] Activities are correct
- [ ] Error handling is in place
- [ ] Logging is included
- [ ] Variable names follow conventions

If needed, ask Autopilot to refine:
```
Add a retry loop around the Document Understanding extraction, max 3 attempts
```

### Step 5: Apply Team Standards

After accepting the workflow:

1. Verify naming conventions
2. Add any missing comments
3. Check error handling patterns
4. Verify logging format
5. Run workflow analyzer

## Tips for Better Prompts

### Be Specific About Types

```
❌ "Create a workflow that processes data"
✅ "Create a workflow with in_Data (DataTable, In) and out_Result (Boolean, Out)"
```

### Specify Modern Design

```
✅ "Use Modern Design activities"
✅ "Use Use Application/Browser activity"
```

### Include Error Handling

```
✅ "Wrap in Try-Catch, catch SelectorNotFoundException and retry up to 3 times"
```

### Reference Existing Patterns

```
✅ "Follow the same pattern as Login_Portal.xaml in this project"
```

## Common Autopilot Prompts

### UI Automation

```
Create a workflow to login to [Application] using Modern Design.
Arguments: in_Credential (SecureCredential, In), out_Success (Boolean, Out)
Use Use Application/Browser, Type Into for username/password, Click for login button.
Verify login success by checking for [element]. Include error handling.
```

### API Integration

```
Create a workflow to call [API endpoint] using HTTP Request.
Arguments: in_RequestBody (String, In), out_Response (JObject, Out)
Include OAuth token refresh if 401 received. Handle rate limiting (429) with retry.
Log request/response at Debug level.
```

### Data Processing

```
Create a workflow to process a DataTable.
Arguments: in_Data (DataTable, In), out_ProcessedData (DataTable, Out)
For each row: validate [fields], transform [logic], add to output.
Skip invalid rows and log them. Return processed DataTable.
```

### Document Understanding

```
Create a workflow to extract data from PDF using Document Understanding.
Arguments: in_FilePath (String, In), out_ExtractedData (DataTable, Out)
Use Digitize Document, then Data Extraction Scope with [ML model].
Extract fields: [field list]. Handle low confidence scores.
```

## Iterating with Autopilot

If the first result isn't right:

```
"Change the variable dt_Temp to dt_ProcessedInvoices"
"Add logging before and after the API call"
"Change the timeout to 60 seconds"
"Add a condition to check if the file exists first"
```

## Using Team Templates

If you have workflow templates:

1. Open the template in Studio
2. Tell Autopilot to use it as reference:
   ```
   Using the ErrorHandling_Template.xaml pattern, create a workflow that...
   ```

## After Generation

1. **Test the workflow** with sample data
2. **Run Workflow Analyzer** to check for issues
3. **Commit to Git repository** (triggers TDD Agent update)

## Troubleshooting

### Autopilot generates wrong activity type

Be more specific:
```
"Use the Modern Design Type Into activity, not the classic one"
```

### Arguments are wrong type

Explicitly state types:
```
"in_Amount should be Decimal type, not String"
```

### Missing error handling

Add to prompt:
```
"Wrap the entire sequence in Try-Catch. Catch Exception, log it, and rethrow."
```

## Best Practices

### Prompt Writing Tips

1. **Be specific about Modern Design**—Autopilot defaults to Modern activities but being explicit helps
2. **Include argument types**—Don't assume Autopilot will infer correctly
3. **Describe error handling upfront**—Easier than adding it after generation
4. **Reference existing workflows**—"Follow the pattern in X.xaml" helps consistency
5. **One workflow at a time**—Generate and verify before moving to the next

### When Autopilot Struggles

Some scenarios where manual work is often needed:

| Scenario | Recommendation |
|----------|----------------|
| Complex selectors | Generate structure, then tune selectors manually |
| Multi-application flows | Break into separate workflows per application |
| Custom activities | Describe desired behavior, add custom activity manually |
| Orchestrator integration | Generate logic, add Orchestrator activities manually |

### Iterative Refinement

Autopilot conversations are stateful—use follow-ups:

```
Initial: "Create a workflow to process invoices..."
Follow-up: "Add a retry loop with 3 attempts around the HTTP Request"
Follow-up: "Change the timeout to 60 seconds"
Follow-up: "Add Info-level logging before each major step"
```

## Related

- [Architecture](../ARCHITECTURE.md) - System overview
- [Spec Agent](../agents/spec/README.md) - How Plan.md is generated
- [UiPath Autopilot Docs](https://docs.uipath.com/autopilot) - Official documentation
