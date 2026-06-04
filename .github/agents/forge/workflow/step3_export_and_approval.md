# Step 3: Export CSV & Get User Approval

## 3.1 Export Process

> **Scope:** Export **manual test cases only** — LLM-evaluated candidates from Step 2. Automated cases (separated in Step 1.5) are excluded from the CSV — they serve as context for generation only.

Create a CSV file in `.mcp/ui/forge_logs/`:

**File naming:** `extraction_<module_name>_<timestamp>.csv`
Example: `extraction_login_module_2026-05-14_103045.csv`

### CSV Columns

| Column       | Description                                      | Source Field                         |
| ------------ | ------------------------------------------------ | ------------------------------------ |
| test_case_id | qTest test case ID                               | id                                   |
| pid          | qTest PID (e.g., TC-1001)                        | pid                                  |
| title        | Test case title                                  | name                                 |
| priority     | Automation priority (HIGH / MEDIUM / LOW / SKIP) | Step 2 evaluation result             |
| link         | Direct URL to test case in qTest                 | Constructed from project.json values |

**Link construction:** Read `qtest_base_url` and `project_id` from `.mcp/automation/project.json` (single source of truth for URL format). Build the URL dynamically — do not hardcode the base URL or project ID here.

### Steps

1. Collect all manual test cases evaluated in Step 2
2. Extract only the five columns above into CSV-compatible format
3. **Set priority:** Use the evaluation result from Step 2 (HIGH / MEDIUM / LOW / SKIP)
4. **Construct link:** Build the qTest URL for each test case using values from project.json
5. Save CSV to `.mcp/ui/forge_logs/`
6. Report to user:
   ```
   ✅ Extracted <count> test cases from qTest
   📄 Saved to: .mcp/ui/forge_logs/extraction_<name>_<timestamp>.csv
   ```

### CSV Format Example

```csv
test_case_id,pid,title,priority,link
100000001,TC-1001,"Verify login with valid credentials",HIGH,https://<your-instance>.qtestnet.com/p/<PROJECT_ID>/portal/project#tab=testdesign&object=1&id=100000001
100000002,TC-1002,"Verify hardware device connection",SKIP,https://<your-instance>.qtestnet.com/p/<PROJECT_ID>/portal/project#tab=testdesign&object=1&id=100000002
```

---

## 3.2 Present Analysis & Get User Approval

**⚠️ CRITICAL: NEVER proceed with generation without user approval.**

### Present Filtering Results

```
🔍 Filtering Analysis Complete
════════════════════════════════════════

Source: qTest Module "<ModuleName>" (ID: <module_id>)
Total Test Cases Retrieved: <count> (manual: <M>, automated: <A> context-only)

📊 Classification Results (manual cases only):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ HIGH Priority (<count> cases):
   → Fully automatable with clear requirements
   → Will generate clean scenarios ready for automation
   [List IDs and titles]

⚠️  MEDIUM Priority (<count> cases):
   → Automatable but needs clarification
   → Will generate drafts with <!-- REVIEW --> markers
   [List IDs, titles, and identified gaps]

❌ LOW Priority (<count> cases):
   → Significant gaps or unclear requirements
   → Require user decision
   [List IDs, titles, and reasoning]

🚫 SKIPPED (<count> cases):
   → Has automation blockers (visual, manual judgment, external deps)
   [List IDs, titles, and reasons]

Context Available:
- Already-automated test cases: <count> (used for generation context in Step 4)
```

**When resuming from CSV** (via Step 1.4): Use the `priority` column directly. LLM reasoning is not available from a previous session — present priorities without gap details for MEDIUM/LOW cases.

### Prompt User for Generation Decision

Ask the user one question:

**Generation Decision:**

- Header: "Generation"
- Question: "Which priority levels should I generate scenarios for?"
- Options:
  - `HIGH priority only` (recommended) — Fully automatable cases
  - `HIGH + MEDIUM priority` — Includes cases needing clarification (will have REVIEW markers)
  - `HIGH + MEDIUM + LOW priority` — Includes cases with significant gaps; LOW cases will have extensive REVIEW markers
  - `Cancel - Just show analysis` — No generation

### Based on User Response

| User Selection                | Action                                                           |
| ----------------------------- | ---------------------------------------------------------------- |
| `Cancel - Just show analysis` | Output analysis summary only, stop execution                     |
| `HIGH priority only`          | Generate HIGH priority scenarios only                            |
| `HIGH + MEDIUM priority`      | Generate HIGH + MEDIUM scenarios                                 |
| `HIGH + MEDIUM + LOW`         | Generate all priorities; LOW cases with extensive REVIEW markers |

**DO NOT automatically proceed without user selection.**
