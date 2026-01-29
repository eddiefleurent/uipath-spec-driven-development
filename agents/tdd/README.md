# TDD Agent

Autonomous agent that updates the Technical Design Document based on workflow changes.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Autonomous |
| **Platform** | UiPath Agent Builder |
| **Input** | Git diff, Plan.md, existing TDD.md |
| **Output** | Updated TDD.md |

## What It Does

1. Receives git diff showing changed XAML files
2. Reads Plan.md to understand what was supposed to be built
3. Reads existing TDD.md to understand current state
4. Generates updated TDD.md with new/changed workflows documented

No conversation needed - give it the inputs, get the updated TDD.md.

## When to Use

After committing workflows to your Git repository:
- New workflows added
- Existing workflows modified
- Workflows deleted

## Input: Getting the Git Diff

Before starting the TDD Agent, get the git diff:

```bash
# For last commit
git diff HEAD~1 -- "*.xaml"

# For a PR/branch
git diff origin/main...HEAD -- "*.xaml"

# Just file names
git diff HEAD~1 --name-only -- "*.xaml"
```

## Building the Agent

### 1. Create Agent Project

```
UiPath Studio → New Project → Agent
Name: TDD_Agent
Type: Autonomous
```

### 2. Configure Settings

| Setting | Value |
|---------|-------|
| Agent Type | Autonomous |
| Temperature | 0.2 (deterministic) |

### 3. System Prompt

Copy the system prompt from [PROMPTS.md](./PROMPTS.md).

Key persona: **Technical Writer** who understands XAML and maintains documentation.

Core objectives:
- Parse git diff to identify changes
- Cross-reference with Plan.md specifications
- Update TDD.md accurately
- Infer purpose from Plan.md and diff context

### 4. Define Tools

| Tool | Purpose |
|------|---------|
| Diff Parser | Extract workflow changes from git diff |
| TDD Updater | Merge changes into existing TDD.md |
| Doc Generator | Format updated TDD.md |

### 5. Processing Flow

```
Input
├── Receive git diff
├── Receive Plan.md
├── Receive existing TDD.md

Analyze
├── Parse changed files from diff
├── Match to Plan.md specifications
├── Identify new/modified/deleted workflows

Generate
├── Update Workflows table
├── Add/update workflow details
├── Update Change Log
└── Output updated TDD.md
```

## Example

**Engineer provides:**
- Git diff showing `ExtractData.xaml` added
- Plan.md with ExtractData spec
- Existing TDD.md

**Agent outputs:**
Updated TDD.md with:
- ExtractData.xaml added to Workflows table
- Full workflow details (arguments, variables, activities from Plan.md)
- Change log entry: "Added ExtractData.xaml"

## Output

Updated TDD.md with:
- New workflows added to Workflows table
- Detailed specs for new/changed workflows
- Change log entry with date

## Related

- [PROMPTS.md](./PROMPTS.md) - Full system prompts and templates
- [TDD Template](../../templates/TDD_TEMPLATE.md) - Full TDD structure
- [Architecture](../../ARCHITECTURE.md) - Overall system design
- [Spec Agent](../spec/README.md) - Creates Plan.md
