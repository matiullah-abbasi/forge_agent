# Step 1: Interactive Setup & Fetch Test Cases

## 1.1 Prompt User for Project and Module Details

Ask the user (single prompt, two questions):

1. **Project ID** — Example: `100001` (from qTest URL: `https://<your-instance>.qtestnet.com/p/<PROJECT_ID>`)
2. **Module Discovery** — Options:
   - `Display All Modules` (recommended)
   - `Enter Specific Module ID`
   - `Enter Module Path`

---

## 1.2 Handle Module Selection

### Option A: Display All Modules

1. Fetch root modules: `search-modules(projectId, parentId=null)`
2. For each root, fetch children: `search-modules(projectId, parentId=<root_id>)`
3. Display ALL modules (root + children) with IDs, names, and full paths.
   **🔴 No silent filtering** — show every module including containers like "Root Test Cases", "Deprecated Test Cases", etc.
   If any child module also has children, indicate with `▸` or `(has sub-modules)` so the user knows to drill down.
4. Prompt user to select module(s) with `multiSelect: true`.

### Option B: Enter Specific Module ID

1. Prompt for module ID.
2. Verify it exists via `search-modules`.

### Option C: Enter Module Path

1. Prompt for path (e.g., `"/<module Path>"`).
2. Search via `search-modules` with module name.
3. If multiple matches — display all, prompt user to pick.

---

## 1.3 Sub-module Handling

Use the module hierarchy already loaded from 1.2 — no additional API calls.

- **No children** → Proceed directly to Resume Check (Step 1.4).
- **Has children** → Prompt user to select sub-module(s) (`multiSelect: true`). Do NOT fetch from all sub-modules automatically.
- If a selected sub-module also has children, apply recursively (prompt again).

---

## 1.4 Resume Check

Once the final module is confirmed (no more sub-module selection needed), check if a previous extraction exists:

1. Search for files matching: `.mcp/ui/forge_logs/extraction_<module_name>_*.csv`
2. If a matching CSV is found:
   - Inform user: "Found previous extraction: `[filename]` ([date])."
   - Ask: "Resume from this CSV (skip fetch + filter) or start fresh?"
     - Options: "Resume from CSV (Recommended)", "Start fresh (re-fetch from qTest)"
   - If "Resume": Load existing CSV, skip directly to Step 3.2 (present analysis & approval).
   - If "Start fresh": Continue to Step 1.5 below.
3. If no CSV found: Continue to Step 1.5 below.

**Note:** When resuming from CSV, the `priority` column is used directly. LLM reasoning from the original run is not available — present priorities without gap details.

---

## 1.5 Fetch Test Cases from qTest

1. Call `list-testcases` for the confirmed module ID(s) only.
2. **Always paginate:** `size=25`, iterate pages until fewer than 25 results returned.

### Separate Already-Automated Cases

Before enrichment, filter out cases where Type = "Automated" (default `field_value = "702"` — may vary by qTest instance):
- Store separately for context enrichment in Step 4
- Report: "Found X already-automated test cases (used for context)"

### Enrichment (MANDATORY)

Call `get-testcase` for **each remaining manual candidate** to retrieve full steps, preconditions, custom fields, and linked requirements. Store enriched data in memory — Step 2 uses this directly, do NOT re-fetch.

### Early-Exit Check

If ALL test cases are Automated:
- Report: "All X test cases in [module] are already automated."
- STOP. Ask user to select a different module or exit.
