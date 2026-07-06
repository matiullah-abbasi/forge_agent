# Step 4: Generate Scenario Files

**⚠️ PREREQUISITE: Only proceed if user approved generation in Step 3 (Generation Decision).**

**Always produce exactly ONE `.md` file per extraction run.** Do NOT create separate files per test case.

---

## 4.1 Gather Context

### Determine Feature Path & Default File Name

The agent already has the full module hierarchy from Step 1 (each module has `name`, `id`, `parent_id`). Use that data directly:

1. Walk the selected module's ancestry chain (via `parent_id`) up to the project root.
2. Skip any top-level container modules that are direct children of the project root and act purely as groupers (e.g., "Root Test Cases"). Keep all other segments.
3. Normalize each kept segment: lowercase, replace spaces and dashes (`-`) with `_`.
4. Join segments with `/` to form `local_path`.
5. `defaultFileName` = last segment of `local_path` + `_flow`.
6. Full output path: `.mcp/ui/scenario_templates/<local_path>/<defaultFileName>.md`

**Example:**
| Module Path | `local_path` | `defaultFileName` |
| ------------------------------------- | ------------------------------------ | --------------------------------- |
| Feature Tests / User Settings | `feature_tests/user_settings` | `user_settings_flow` |
| Feature Tests / Download Report | `feature_tests/download_report` | `download_report_flow` |

### Read Test Scenario Template (MANDATORY)

Load `.mcp/ui/scenario_templates/test_scenario_template.md` — this is the TEST SCENARIO TEMPLATE. Copy its EXACT format for all scenario generation.

### Read Feature-Specific Examples

Search in: `.mcp/ui/scenario_templates/<local_path>/` (using `local_path` derived above).

- If `.md` files exist in that directory: read 1–2 to extract feature-specific UI element patterns and phrasing.
- If no examples exist: rely on the test scenario template and qTest module description.

---

## 4.2 Resolve File Name

### 4.2a Prompt User to Confirm or Customise

**🔴 MANDATORY — Do not proceed to file creation without user confirmation.**

Ask the user **once**:

- Header: "Scenario File Name"
- Question: "The scenario file will be saved as `<defaultFileName>`. Confirm or enter a custom name."
- Context: "File will be saved to: `.mcp/ui/scenario_templates/<local_path>/` | Must end with `_flow.md`"
- Options:
  - `Use default: <defaultFileName>` (recommended)
  - `Enter a custom file name`

If custom name entered:

- Append `_flow.md` automatically if omitted
- Replace spaces with underscores, lowercase the result

### 4.2b Handle Existing File

If a file already exists at the target path:

- Inform user: "Overwriting existing file: `<filename>`"
- Overwrite it. This workflow is idempotent — re-running on the same module replaces the previous output.

---

## 4.3 Generate Scenario Content

### Test Case Ordering (MANDATORY)

**Steps MUST follow the qTest `order` field sequence.** Test cases are already sorted by `order` ascending from Step 1.5 — verify the order is intact and do NOT regroup or reorder by topic, similarity, or any other logic. The qTest order reflects the intended test execution sequence and must be preserved exactly. If the order was lost (e.g., resuming from CSV), re-sort by `order` ascending as a safety net.

For test cases where the user provided additional details in Step 2, incorporate that context into preconditions, steps, or expected results as appropriate.

Write clean steps for each test case — no `<!-- REVIEW -->` markers or annotations.

### Formatting

Apply formatting rules from: `draft_scenarios/references/formatting_patterns.md`

That file is the **single source of truth** for titles, objectives, preconditions, step formatting, UI element patterns, and post-execution blocks. If it does not exist or fails to load, fall back to the test scenario template.

---

## 4.4 Prepend Extraction Metadata

At the very top of the file, add:

```markdown
<!-- extraction_metadata: qtest_ids=[<id1>,<id2>] | module=<qtest_module_path> | extracted=<YYYY-MM-DD> -->
```

- `qtest_ids`: comma-separated list of all test case IDs included in this file
- `module`: full qTest module path (e.g., `<Root Module>/Login/Authentication`)
- `extracted`: today's date

---

## 4.5 Write File

Save the file to: `.mcp/ui/scenario_templates/<local_path>/<resolvedFileName>`

---
