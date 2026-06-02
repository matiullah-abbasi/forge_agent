# Scenario Formatting Patterns

**MANDATORY: Follow these exact patterns from existing scenario files.**

---

## File Naming Convention

Group test cases by functionality and name files descriptively:

1. Identify common functionality across test cases
2. Convert to snake_case, MUST end with `_flow.md`, keep under 50 characters

| Test Cases                                   | Functionality Group   | Filename                           |
| -------------------------------------------- | --------------------- | ---------------------------------- |
| Create project, Edit project, Delete project | Project Management    | `project_management_flow.md`       |
| Rename item, Refresh item, Delete item        | Item Operations       | `item_operations_flow.md`          |
| Stop during streaming, Query preservation    | Stop Response         | `stop_response_generation_flow.md` |
| Download as PDF, Download as CSV             | Download Data         | `download_data_flow.md`            |

---

## Title Pattern

> **⚠️ qTest IDs must NEVER appear inside scenario blocks — headings, body lines, or anywhere else. They belong only in the `extraction_metadata` comment at the top of the file.**

```markdown
# Test Scenario — <Name> Flow
```

Examples:

- `# Test Scenario — Create New Project Flow`
- `# Test Scenario — Download Report Flow`
- `# Test Scenario — Stop Response Generation Flow`

---

## Objective Pattern

Always starts with: **"Automate the flow where a user..."**

```markdown
Automate the flow where a user creates a new project via **Project Listing** listing button,
uses the **Search Panel** via the **"+" icon** to generate a result from a query, adds the
result to the project and performs different operations with the items.
```

---

## File Structure Rules

- All steps in a single flat numbered list — no `###` subsection headings, no `## Scenario N:` blocks.
- Step numbers are continuous from 1 to N — never reset.
- Common setup steps (open browser, navigate, login) appear once at the start.
- `## Objective`, `## Preconditions`, `## Steps`, and `## Post-Execution` each appear exactly once per file.

---

## Steps Pattern

**Always start with these 3 steps:**

```markdown
1. **Open browser.**
2. **Navigate** to the application URL based on the environment variable and credentials.
3. **Login** using valid credentials.
```

**Formatting Rules by Element:**

| Element Type | Format                 | Example                                  |
| ------------ | ---------------------- | ---------------------------------------- |
| Buttons      | **Button Name** button | Click the **Submit** button              |
| Icons        | **"Icon Name"**        | Open search panel using **"+" icon**    |
| Fields       | **Field Name**         | Enter text in the **Input field**        |
| Menus        | **Menu Name**          | Click **three dots**/**kebab** button    |
| Modals       | **Modal Name**         | Open **Project Listing** modal           |
| Queries/Text | Backticks              | `Show items in my project`               |
| UI Labels    | **Label Name**         | Verify **Item Title** appears            |
| Wait Actions | Wait until/for         | Wait until the **response is generated** |
| Verification | Verify that...         | Verify that **Input Field** expands      |

**✅ GOOD examples:**

```markdown
4. Open search panel using **"+" icon**.
5. Enter the following query in the input field:
   `Show active items in the project in form of chart`
6. Press **Enter** to send the query.
7. Wait until the **response is generated**.
8. Click **Download Button** to download the data
9. Hover over the first item
10. Verify that **Input Field** expands when user enters text
11. Click **three dots**/**kebab** button in the item
```

**❌ BAD examples (DO NOT DO THIS):**

```markdown
4. page.getByTestId('search-panel-icon').click()
5. await fillInput(page, locator, query)
6. Click the button (which button?)
7. Verify it works (verify what exactly?)
```
