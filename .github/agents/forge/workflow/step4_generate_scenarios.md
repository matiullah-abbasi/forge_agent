# Step 4: Generate Scenario Files

**⚠️ PREREQUISITE: Only proceed if user approved generation in Step 3 (Generation Decision).**

**Always produce exactly ONE `.md` file per extraction run.** Do NOT create separate files per priority level or per test case. Order scenarios within the file: HIGH → MEDIUM → LOW.

---

## 4.1 Gather Context

### Determine Feature Path

The agent already has the full module hierarchy from Step 1 (each module has `name`, `id`, `parent_id`). Use that data directly:

1. Walk the selected module's ancestry chain (via `parent_id`) up to the project root.
2. Skip any top-level container modules that are direct children of the project root and act purely as groupers (e.g., "Root Test Cases"). Keep all other segments.
3. Normalize each kept segment: lowercase, replace spaces and dashes (`-`) with `_`.
4. Join segments with `/` to form `local_path`.
5. `feature_name` = the original name of the selected module (un-normalized).

### Read Master Template (MANDATORY)

Load the **master template** from the path configured in `project.json` under `"master_template"`. Copy its EXACT format for all scenario generation.

> **Setup:** Create a master template file (e.g., `.mcp/ui/scenario_templates/test_scenario_template.md`) that defines the canonical structure: title, objective, preconditions, steps, and post-execution. Set its path in `project.json` so the agent knows where to find it.

### Read Feature-Specific Examples

Search in: `.mcp/ui/scenario_templates/<local_path>/` (using `local_path` derived above).

- If `.md` files exist in that directory: read 1–2 to extract feature-specific UI element patterns and phrasing.
- If no examples exist: rely on the master template and qTest module description.

---

## 4.2 Resolve File Name

### 4.2a Derive Default Name from qTest Module Name

Convert the **qTest module name** to a snake_case filename:

Rules:

- Lowercase all words
- Replace spaces and special characters with underscores `_`
- Strip leading/trailing underscores
- Always append `_flow.md`

**Example:**
| qTest Module Name | Default File Name |
| ------------------------- | --------------------------------- |
| `User Settings`          | `user_settings_flow.md`           |
| `Download Report`        | `download_report_flow.md`         |

The feature directory uses `local_path` derived in 4.1.
Full output path: `.mcp/ui/scenario_templates/<local_path>/<filename>`

### 4.2b Prompt User to Confirm or Customise

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

### 4.2c Handle Existing File

If a file already exists at the target path:

- Inform user: "Overwriting existing file: `<filename>`"
- Overwrite it. This workflow is idempotent — re-running on the same module replaces the previous output.

---

## 4.3 Generate Scenario Content

Write scenarios in this order:

1. **HIGH priority** — clean steps
2. **MEDIUM priority** — clean steps (same formatting as HIGH)
3. **LOW priority** (only if "All priorities" was selected) — clean steps (same formatting as HIGH)

**🔴 IMPORTANT: Do NOT add extensive review-point comments (e.g., `<!-- REVIEW: reason -->` or `<!-- DRAFT: explanation -->`). For MEDIUM and LOW priority test cases, add only a single `<!-- REVIEW Required -->` comment above their first step. No inline review comments within steps. All steps themselves must be written cleanly with no annotations.**

### Formatting

Apply formatting rules from: `forge/reference/formatting_patterns.md`

That file is the **single source of truth** for titles, objectives, preconditions, step formatting, UI element patterns, and post-execution blocks. If it does not exist or fails to load, fall back to the master template.

### Priority Handling

**If user selected "HIGH priority only":**
Include only HIGH-priority cases.

**If user selected "HIGH + MEDIUM priority":**
Include HIGH and MEDIUM cases — all written as clean steps.

**If user selected "HIGH + MEDIUM + LOW priority":**
Include all non-skipped cases — all written as clean steps.

---

## 4.4 Prepend Extraction Metadata

At the very top of the file, add:

```markdown
<!-- extraction_metadata: qtest_ids=[<id1>,<id2>] | module=<qtest_module_path> | extracted=<YYYY-MM-DD> | priority_levels=<HIGH|HIGH+MEDIUM|ALL> -->
```

- `qtest_ids`: comma-separated list of all test case IDs included in this file
- `module`: full qTest module path (e.g., `<Root Module>/Login/Authentication`)
- `extracted`: today's date
- `priority_levels`: which levels were generated

---

## 4.5 Write File

Save the file to: `.mcp/ui/scenario_templates/<local_path>/<resolvedFileName>`

---
