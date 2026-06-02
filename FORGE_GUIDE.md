# qTest to Playwright Extraction System — Complete Guide

## Overview

Automated bridge between qTest manual test cases and Playwright test automation. Extracts test cases from qTest, filters for automation suitability using LLM analysis, and generates standardized Markdown scenario templates ready for Playwright test generation.

**Design Principle:** Keep it simple. No over-engineering.

---

## Problem Statement

**Gap:** Manual extraction layer between qTest (test management) and Playwright (test execution)

**Current State:**

- Manual test cases exist in qTest
- Requires human intervention to filter, format, and convert them into automation-ready templates

**Solution:** Automated `@forge` agent

---

## Quick Start

### 1. Extract Test Cases

```
@forge Extract test cases from qTest module "/<Module Path>"
```

Or by module ID:

```
@forge Extract test cases from qTest module <MODULE_ID>
```

### 2. Review Generated Scenarios

| Priority | Location                                | Status          | Action                                |
| -------- | --------------------------------------- | --------------- | ------------------------------------- |
| HIGH     | `.mcp/ui/scenario_templates/<feature>/` | ✅ Ready        | Use with Playwright agent directly    |
| MEDIUM   | `.mcp/ui/scenario_templates/<feature>/` | ⚠️ Needs review | Resolve `<!-- REVIEW -->` annotations |
| LOW/SKIP | `.mcp/manual_candidates/report.md`      | ❌ Not suitable | Review blockers manually              |

### 3. Generate Playwright Tests

```
@playwright Generate test from .mcp/ui/scenario_templates/<feature>/<desired_md_file> for <desired_env>
```

---

## Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                        EXTRACTION AGENT                             │
│                     (GitHub Copilot Agent)                          │
└────────────────────────────────────────────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
   │  qTest Reader   │  │  Filter Engine  │  │  MD Generator   │
   │     Module      │  │  (2-Step Filter)│  │     Module      │
   └─────────────────┘  └─────────────────┘  └─────────────────┘
           │                     │                     │
           ▼                     ▼                     ▼
   [qTest MCP API]      [Config Rules]         [.md Templates]
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
         ┌─────────┐       ┌─────────┐        ┌─────────┐
         │  HIGH   │       │ MEDIUM  │        │  LOW/   │
         │ Priority│       │ Priority│        │  SKIP   │
         └─────────┘       └─────────┘        └─────────┘
              │                  │                  │
              ▼                  ▼                  ▼
   scenario_templates/    scenario_templates/   manual_candidates/
   <feature>/             <feature>/            report.md
                          (with REVIEW markers)
```

---

## Components

### 1. qTest Reader Module

Fetches test cases from qTest and maps them to the local folder structure.

- `getModuleTree(projectId)` — Recursively fetch qTest module hierarchy
- `mapModuleToPath(modulePath)` — Convert qTest module path to local folder path
- `getTestCasesForModule(moduleId)` — Fetch all test cases in a module
- `enrichTestCase(testCaseId)` — Get full test case details (steps, expected results, properties)

**Dependencies:** qTest MCP tools (`list-testcases`, `get-testcase`, `search-modules`), project ID from `project.json`

---

### 2. Filter Engine — LLM Evaluation

#### LLM Analysis

Intelligent analysis of manual candidates using LLM.

**LLM evaluates:**

1. **Automation Suitability** — Can this be automated? (identifies blockers: visual validation, manual judgment, external dependencies)
2. **Test Case Clarity** — Are requirements clear enough? (title, description, preconditions, steps)
3. **Priority Classification:**

| Priority   | Condition                                    | Output                                  |
| ---------- | -------------------------------------------- | --------------------------------------- |
| **HIGH**   | Fully automatable with clear requirements    | Clean scenario file                     |
| **MEDIUM** | Automatable but needs clarification          | Scenario with `<!-- REVIEW -->` markers |
| **LOW**    | Significant gaps or unclear requirements     | User decides; draft or skip             |
| **SKIP**   | Has automation blockers or already automated | Logged to `manual_candidates/report.md` |

---

### 3. MD Generator Module

Converts filtered test cases into standardized `.md` scenario templates.

| qTest Field    | Template Section    | Notes                             |
| -------------- | ------------------- | --------------------------------- |
| Test Case Name | Title (H1)          | Prefixed with "Test Scenario — "  |
| Description    | Objective           | High-level goal                   |
| Preconditions  | Preconditions       | Template defaults + test-specific |
| Test Steps     | Steps (numbered)    | Actions + expected results        |
| Module Path    | Directory structure | Determines output file path       |

---

### 4. Context Enrichment

Enhances generated scenarios with project-specific knowledge from:

1. `.mcp/project_context.md` — Product features, terminology, user roles
2. `.mcp/ui/scenario_templates/**/*.md` — Patterns from existing scenarios
3. qTest custom fields — Tags, priority, linked requirements

---

## Configuration Files

All configuration lives in `.mcp/ui/automation_config/`:

### `project.json`

Single source of truth for the qTest project ID and base URL. Users can override the project ID at runtime via interactive prompt.

```json
{
  "project_id": "<YOUR_PROJECT_ID>",
  "qtest_base_url": "https://<your-instance>.qtestnet.com",
  "master_template": ".mcp/ui/scenario_templates/test_scenario_template.md"
}
```

- `master_template` — Path to your master scenario template file. Create this file with your desired scenario structure (title, objective, preconditions, steps, post-execution). The agent copies its exact format when generating scenarios.

---

## Directory Structure

```
.mcp/
├── ui/
│   ├── automation_config/           # Configuration files
│   │   └── project.json                # Project ID — single source of truth
│   ├── scenario_templates/          # ✅ HIGH + MEDIUM priority outputs
│   │   ├── test_scenario_template.md   # Master template (user-created)
│   │   ├── <feature_a>/
│   │   ├── <feature_b>/
│   │   └── ...
│   └── forge_logs/                  # Execution logs + CSV exports
│       └── extraction_<module>_<timestamp>.csv
├── manual_candidates/              # ❌ LOW/SKIP aggregated report
│   └── report.md
└── project_context.md              # Product domain knowledge for step enrichment
```

---

## Workflow — Step by Step

```
qTest Project
     │
     │ qTest MCP API
     ▼
Fetch Test Cases (by module ID or path)
     │
     ├── Type = Automated → separated for context only
     └── Manual cases → proceed to LLM
     │
     ▼
LLM Analysis
     │
     ├── HIGH  → scenario_templates/<feature>/<name>_flow.md
     ├── MEDIUM → scenario_templates/<feature>/<name>_flow.md  (with REVIEW markers)
     ├── LOW   → user decides (draft or skip)
     └── SKIP  → manual_candidates/report.md
```

---

## Command Reference

| Command                           | Purpose                            | Output                       |
| --------------------------------- | ---------------------------------- | ---------------------------- |
| `@forge Extract from "/<Module Path>"` | Extract all test cases from module | Scenarios in `.mcp/` folders |
| `@forge Extract from <MODULE_ID>`       | Extract by module ID               | Scenarios in `.mcp/` folders |

---

## Usage Examples

### Example 1: Extract a specific module

```
@forge Extract test cases from qTest module "/<Parent Module>/<Child Module>"
```

Output:

```
✅ HIGH:     8 scenarios → .mcp/ui/scenario_templates/<parent_module>/<child_module>/
⚠️  MEDIUM:  3 drafts   → .mcp/ui/scenario_templates/<parent_module>/<child_module>/ (with REVIEW markers)
❌ SKIP:     2 logged   → .mcp/manual_candidates/report.md
```

### Example 2: HIGH priority scenario output

```markdown
# Test Scenario — Download Data as PDF

## Objective

Automate the flow where a user downloads data as a PDF...

## Preconditions

- A valid user account exists...

## Steps

1. **Open browser.**
2. **Navigate** to the application URL...
3. **Login** using valid credentials.
4. Click **Download Button** to download the data.
```

### Example 3: MEDIUM priority draft output

```markdown
## Steps

... 6. Edit the **item title** to a new value.

<!-- REVIEW: Confirm exact element — is it an inline edit field or a modal? -->

7. Verify the title is updated.
<!-- REVIEW: Check if title text matches the new value in the item header -->
```

---

## Troubleshooting

| Problem                         | Solution                                                  |
| ------------------------------- | --------------------------------------------------------- |
| No test cases found             | Verify module ID/path in qTest                            |
| All marked as SKIP              | Adjust LLM evaluation criteria or check test case content |
| qTest connection failed         | Check MCP server and credentials                          |
| Duplicate filenames             | Agent auto-appends `_v2`, `_v3`                           |
| Module has no direct test cases | Agent prompts to select a submodule                       |

---

## Maintenance

| What to change                     | Where                                                  |
| ---------------------------------- | ------------------------------------------------------ |
| Step-by-step workflow instructions | `forge/workflow/step<N>_*.md`                          |
| Formatting rules                   | `forge/reference/formatting_patterns.md`               |
