# Step 2: Filter Test Cases

## 2.1 Evaluate Candidates for Automation Suitability

**Purpose:** Analyze each manual candidate test case (from Step 1.5) and classify by automation priority.

⛔ Do NOT make any qTest API calls in this step. All test case data was already fetched during Step 1.5 enrichment. Use only the data stored in memory.

### Evaluation Criteria

For each candidate test case, evaluate:

#### 1. Automation Suitability

Can this be automated with Playwright?

- **Blockers to identify:**
  - Visual validation (e.g., "looks correct", "appears properly", "visually verify")
  - Manual judgment (e.g., "manually confirm", "exploratory testing", "usability assessment")
  - External dependencies (hardware, external tools, manual setup)

#### 2. Test Case Clarity

Are requirements clear enough to automate?

- Analyze title, description, preconditions, and steps
- Identify missing or ambiguous information
- Determine if test intent is clear despite missing details

#### 3. Priority Classification

| Priority   | Condition                                                        |
| ---------- | ---------------------------------------------------------------- |
| **HIGH**   | Fully automatable with clear requirements                        |
| **MEDIUM** | Automatable but needs clarification or has minor gaps            |
| **LOW**    | Has significant gaps or unclear requirements                     |
| **SKIP**   | Has automation blockers (visual, manual judgment, external deps) |

### Process

For each candidate:

1. Review the test case title, description, preconditions, and steps
2. Identify any automation blockers from the criteria above
3. Assess clarity of requirements
4. Assign a priority (HIGH / MEDIUM / LOW / SKIP)
5. Note reasoning: what blockers exist, what's unclear, what's missing
6. Store the result (priority + reasoning) against the test case ID

### Important Notes

- Missing steps are NOT automatic blockers if the title clearly conveys test intent
- Focus on automation feasibility, not test case completeness
- A test case with a clear title but empty steps can still be HIGH priority
- Use `project_context.md` (loaded during Initialization) for domain understanding
