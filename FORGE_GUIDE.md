# qTest to Playwright Extraction System — Complete Guide

## Overview

Automated bridge between qTest manual test cases and Playwright test automation. Extracts test cases from qTest, filters for automation suitability , and generates standardized Markdown scenario templates ready for Playwright test generation.

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
| LOW      | `.mcp/ui/scenario_templates/<feature>/` | ⚠️ Needs review | Only if user opts to include          |
| SKIP     | `.mcp/ui/forge_logs/extraction_*.csv`   | ❌ Not suitable | Logged in CSV with reason; review blockers manually |

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
   │     Module      │  │  (LLM Analysis) │  │     Module      │
   └─────────────────┘  └─────────────────┘  └─────────────────┘
           │                     │                     │
           ▼                     ▼                     ▼
   [qTest MCP API]      [Config Rules]         [.md Templates]
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
         ┌─────────┐       ┌─────────┐        ┌─────────┐
         │  HIGH   │       │ MEDIUM  │        │  SKIP   │
         │ Priority│       │ Priority│        │         │
         └─────────┘       └─────────┘        └─────────┘
              │                  │                  │
              ▼                  ▼                  ▼
   scenario_templates/    scenario_templates/   forge_logs/
   <feature>/             <feature>/            extraction_*.csv
   (clean scenarios)      (with REVIEW marker)  (logged with priority=SKIP)
```

> **Note:** SKIP cases are NOT written to a separate report file. They are tracked in the CSV export (`priority=SKIP` column) and displayed in the Step 3 analysis summary with their blocker reasons. The CSV is the single source of truth for all classification decisions.

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

| Priority   | Condition                                    | Output                                             |
| ---------- | -------------------------------------------- | -------------------------------------------------- |
| **HIGH**   | Fully automatable with clear requirements    | Clean scenario file                                |
| **MEDIUM** | Automatable but needs clarification          | Scenario with `<!-- REVIEW Required -->` marker    |
| **LOW**    | Significant gaps or unclear requirements     | User decides; include in scenario file or exclude  |
| **SKIP**   | Has automation blockers or already automated | Logged in CSV export (`priority=SKIP`)             |

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
  "qtest_base_url": "https://<your-instance>.qtestnet.com"
}
```

---

## Directory Structure

```
.mcp/
├── ui/
│   ├── automation_config/           # Configuration files
│   │   └── project.json                # Project ID — single source of truth
│   ├── scenario_templates/          # ✅ HIGH + MEDIUM (+ LOW if selected) priority outputs
│   │   ├── <feature_a>/
│   │   ├── <feature_b>/
│   │   ├── <feature_c>/
│   │   └── ...
│   ├── FORGE_GUIDE.md
│   ├── forge_logs/                  # Extraction logs + CSV exports (includes SKIP cases)
│   │   └── extraction_<module>_<timestamp>.csv
│   └── ui_test_generator.md
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
     ├── HIGH   → scenario_templates/<feature>/<name>_flow.md
     ├── MEDIUM → scenario_templates/<feature>/<name>_flow.md (with REVIEW marker)
     ├── LOW    → user decides: include in scenario file or exclude
     └── SKIP   → logged in CSV export (forge_logs/extraction_*.csv)
                   displayed in analysis summary with blocker reasons
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
⚠️  MEDIUM:  3 drafts   → .mcp/ui/scenario_templates/<parent_module>/<child_module>/ (with REVIEW marker)
🚫 SKIP:     2 logged   → .mcp/ui/forge_logs/extraction_<module>_<timestamp>.csv (priority=SKIP)
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
