# Step 3: Export CSV & Get User Approval

## 3.1 Export Process

> **Scope:** Export all test cases that passed Step 2 filtering (✅ PASS or ⚠️ CLARIFIED). Automated cases (separated in Step 1.5) are excluded from the CSV — they serve as context for generation only.

Create a CSV file in `.mcp/qtest_extracted_test_cases/`:

**File naming:** `<module_name>_<timestamp>.csv`
Example: `login_module_2026-05-14_103045.csv`

### CSV Columns

| Column       | Description                 | Source Field                         |
| ------------ | --------------------------- | ------------------------------------ |
| test_case_id | qTest test case ID          | id                                   |
| pid          | qTest PID (e.g., TC-1001)  | pid                                  |
| title        | Test case title             | name                                 |
| link         | Direct URL to test case     | Constructed from project.json values |

**Link construction:** Read `qtest_base_url` and `project_id` from `.mcp/automation/project.json` (single source of truth for URL format). Build the URL dynamically — do not hardcode the base URL or project ID here.

### Steps

1. Collect all test cases that passed Step 2 (✅ PASS or ⚠️ CLARIFIED) — **preserve the qTest `order` sort established in Step 1.5**
2. Extract only the four columns above into CSV-compatible format
3. Save CSV to `.mcp/qtest_extracted_test_cases/`
4. Report to user:
   ```
   ✅ Extracted <count> test cases from qTest
   📄 Saved to: .mcp/qtest_extracted_test_cases/<name>_<timestamp>.csv
   ```

### CSV Format Example

```csv
test_case_id,pid,title,link
100000001,TC-1001,"Verify login with valid credentials",https://<your-instance>.qtestnet.com/p/<PROJECT_ID>/portal/project#tab=testdesign&object=1&id=100000001
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

📊 Final Results:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Ready for Generation (<count> cases):
   [List IDs and titles]

🚫 Skipped (<count> cases):
   [List IDs, titles, and reasons]

Context Available:
- Already-automated test cases: <count> (used for generation context in Step 4)
```

**When resuming from CSV** (via Step 1.4): All cases in the CSV are ready for generation.

---

### Prompt User for Generation Decision

Ask the user one question:

**Generation Decision:**

- Header: "Generation"
- Question: "Proceed with scenario generation for all remaining test cases?"
- Options:
  - `Generate All` (recommended) — Generate scenarios for all test cases
  - `Cancel` — No generation, just show analysis

### Based on User Response

| User Selection | Action                                          |
| -------------- | ----------------------------------------------- |
| `Generate All` | Generate scenarios for all remaining test cases |
| `Cancel`       | Output analysis summary only, stop execution    |

**DO NOT automatically proceed without user selection.**
