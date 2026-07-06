# UI Automation Scenarios Generator

Extracts manual test cases from qTest, filters for automation suitability, and generates standardized Playwright scenario templates.

---

## What It Does

- **Silently loads config** on startup (default project ID + product context) — no setup needed from you
- **Fetches test cases from qTest** for the module you choose, paginated to avoid overflows
- **Evaluates each test case** using LLM reasoning — only automation-suitable cases are kept
- **Exports a CSV** of the filtered list and waits for your approval before writing any files
- **Generates scenario `.md` files** in `feature_flows/`, ready for the UI Test Generator agent
- **Read-only against qTest** — never modifies test cases, idempotent on re-runs

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        VS Code IDE                           │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │          UI Automation Scenarios Generator             │  │
│  │                                                        │  │
│  │  ┌──────────────────────┐   ┌───────────────────────┐  │  │
│  │  │  4-Step Workflow      │   │  LLM Filtering Engine │  │  │
│  │  │                      │   │                       │  │  │
│  │  │  1. Interactive setup│   │  • Automation         │  │  │
│  │  │  2. Fetch & filter   │   │    suitability check  │  │  │
│  │  │  3. CSV export +     │   │  • Deduplication      │  │  │
│  │  │     user approval    │   │  • Formatting rules   │  │  │
│  │  │  4. Generate .md     │   │                       │  │  │
│  │  └──────────────────────┘   └───────────────────────┘  │  │
│  └────────────────────────────────────────────────────────┘  │
│                              │                               │
└──────────────────────────────│───────────────────────────────┘
                               │
                  ┌────────────▼────────────┐
                  │    qTest MCP Server      │──────► qTest API
                  └─────────────────────────┘
```

---

## How to Run

1. Open **GitHub Copilot Chat** in VS Code (`Ctrl+Shift+I`)
2. Select **UI Automation Scenarios Generator** from the agents dropdown
3. No arguments needed — the agent starts immediately and guides you

The agent will interactively ask for:

| Prompt         | Details                                                                       |
| -------------- | ----------------------------------------------------------------------------- |
| **Project ID** | qTest project to extract from — a default is pre-configured, you can override |
| **Module**     | qTest module path to target (agent lists options if ambiguous)                |
| **Approval**   | Review the filtered CSV before any `.md` files are written                    |

> No terminal commands needed. All interaction happens inside the chat.

---

## Output

| File                                    | Location                                                |
| --------------------------------------- | ------------------------------------------------------- |
| `<feature>_flow.md` (scenario template) | `.github/agents/ui_automation/feature_flows/<feature>/` |
