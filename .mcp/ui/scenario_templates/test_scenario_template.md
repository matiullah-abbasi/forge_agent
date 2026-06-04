# Test Scenario — <FeatureName> Flow

## Objective

Automate the flow where a user [describe the high-level user journey being tested, including key UI elements and actions].

## Preconditions

- A valid user account exists with sufficient permissions to access the application
  features under test.

- Test credentials for the user account are available and valid for the selected
  environment.

- The target environment is explicitly provided via the `ENV` key and can be
  resolved to a base URL using `main/credentials/environments.json`.

- The application is reachable at the resolved base URL and is in a stable state
  (for example: no maintenance mode, blocking modals, or forced onboarding flows).

- Any feature flags, roles, or configuration required for the scenario are already
  enabled for the user account.

- The user is not authenticated at the start of the test unless explicitly stated
  otherwise in the scenario steps.

- No assumptions are made about the initial UI or feature state unless explicitly
  stated in the scenario steps.

- Add new locators/actions/assertions/tasks in main/ui/<featureName>/<subFeatureName> directory

## Steps

1. **Open browser.**
2. **Navigate** to the application URL based on the environment variable and credentials.
3. **Login** using valid credentials.
4. [Continue with feature-specific steps...]

## Post-Execution

Once the test scenario/spec is written and saved, execute it with the environment provided in your Copilot prompt/context.

**Command (PowerShell):**

```powershell
$env:ENV="<environment>"; npx playwright test tests/ui/<FEATURE_NAME>/<FEATURE_FILE_NAME>.spec.ts
```

> Replace `<environment>` with the environment you supply in your prompt.
> Replace the spec file path with the one which you wrote the test.
> Execute the command.
