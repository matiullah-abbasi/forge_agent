# Forge Agent — qTest to Playwright Automation Bridge

A GitHub Copilot custom agent that extracts manual test cases from qTest, filters them for automation suitability using LLM analysis, and generates standardized Markdown scenario templates ready for Playwright test generation.

---

## Prerequisites

- **VS Code** with GitHub Copilot (agent mode enabled)
- **qTest MCP Server** configured and running — provides API access to your qTest instance
- **qTest account** with access to the target project's test cases

---

## Installation

1. **Copy agent files** into your workspace under `.github/agents/`:

   ```
   your-project/
   ├── .github/
   │   └── agents/
   │       ├── forge.agent.md          ← Agent definition
   │       └── forge/                  ← Workflow & reference files
   │           ├── reference/
   │           │   └── formatting_patterns.md
   │           └── workflow/
   │               ├── step1_interactive_setup.md
   │               ├── step2_fetch_and_filter_test_cases.md
   │               ├── step3_export_and_approval.md
   │               └── step4_generate_scenarios.md
   ```

   > **Note:** This repo contains the source files. Copy `forge.agent.md` and the `forge/` directory into `.github/agents/` in your target project. The `FORGE_GUIDE.md` and `README.md` are documentation only — they don't need to be copied.

2. **Create the configuration directory** in your workspace root:

   ```
   .mcp/
   └── ui/
       └── automation_config/
           └── project.json
   ```

3. **Configure `project.json`:**

   ```json
   {
     "project_id": "<YOUR_QTEST_PROJECT_ID>",
     "qtest_base_url": "https://<your-instance>.qtestnet.com",
     "master_template": ".mcp/ui/scenario_templates/test_scenario_template.md"
   }
   ```

   - `project_id` — Your qTest project ID (visible in the qTest URL)
   - `qtest_base_url` — Your qTest instance URL
   - `master_template` — Path to your scenario template file (defines the output format for generated scenarios)

4. **Create `project_context.md`** (optional but recommended):

   ```
   .mcp/
   └── project_context.md
   ```

   Add product-specific context: feature names, user roles, terminology, UI conventions. This helps the agent generate more accurate scenario steps.

---

## Usage

In VS Code Copilot Chat (agent mode):

```
@forge Extract test cases from qTest module "/<Module Path>"
```

Or by module ID:

```
@forge Extract test cases from qTest module <MODULE_ID>
```

The agent will interactively guide you through module selection, filtering, and scenario generation.

---

## Output Structure

```
.mcp/
└── ui/
    ├── scenario_templates/          # Generated scenario files
    │   └── <feature>/
    │       └── <name>_flow.md
    └── forge_logs/                  # Extraction logs + CSV exports
        └── extraction_<module>_<timestamp>.csv
```

> **SKIP cases** are recorded in the CSV export (with `priority=SKIP`) and displayed during the analysis summary in Step 3. No separate report file is generated — the CSV serves as the single source of truth for all classification decisions.

---

## How It Works

1. **Fetch** — Pulls test cases from qTest via MCP tools
2. **Filter** — LLM evaluates each test case for automation suitability (HIGH / MEDIUM / LOW / SKIP)
3. **Export** — Saves classification to CSV (including SKIP cases with reasons), presents analysis for user approval
4. **Generate** — Creates formatted `.md` scenario templates from approved cases

See [FORGE_GUIDE.md](FORGE_GUIDE.md) for the full technical reference.

---

## Customization

| What to change | Where |
|---|---|
| Workflow logic | `forge/workflow/step<N>_*.md` |
| Formatting rules | `forge/reference/formatting_patterns.md` |
| Agent behavior | `forge.agent.md` |

---

## License

MIT — see [LICENSE](LICENSE)
