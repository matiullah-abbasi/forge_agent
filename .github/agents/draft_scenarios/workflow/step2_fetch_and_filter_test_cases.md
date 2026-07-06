# Step 2: Filter Test Cases

## 2.1 Evaluate Candidates for Automation Suitability

**Purpose:** Analyze each manual candidate test case (from Step 1.5), identify automation blockers, clarify ambiguous cases with the user, and determine which test cases proceed to Step 3.

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

### Process

For each candidate:

1. Review the test case title, description, preconditions, and steps
2. Identify any automation blockers from the criteria above
3. Assess clarity of requirements
4. For test cases whose requirements are unclear or ambiguous (missing steps, vague expected results, undefined UI elements), prompt the user to provide clarity using `vscode/askQuestions` — one prompt per unclear test case:

   ```
   [PID] "<title>"
   Issue: <what is unclear/missing>

   Options:
   - Provide additional details (type your clarification below)
   - No change — proceed with existing info
   - Skip — remove from flow
   ```

   Based on user response:
   - **User provides details:** Store the clarification against the test case and mark as ⚠️ CLARIFIED.
   - **User selects "No change":** Mark as ✅ PASS and proceed with existing data as-is.
   - **User selects "Skip":** Mark as 🚫 SKIPPED and remove from flow entirely.

5. For clear, blocker-free test cases → mark as ✅ PASS (no user prompt needed)

### Important Notes

- Missing steps are NOT automatic blockers if the title clearly conveys test intent
- Focus on automation feasibility, not test case completeness
- A test case with a clear title but empty steps can still pass
- Use `project_context.md` (loaded during Initialization) for domain understanding
