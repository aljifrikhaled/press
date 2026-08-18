# Testing

Press utilizes both backend and frontend test suites.

## Backend Tests (Python/Frappe)

Backend tests are written using Frappe's testing framework (which uses Python's `unittest` module).

*   **Location:** Found in the `tests/` directory of various DocTypes (e.g., `press/press/doctype/site/test_site.py`).
*   **Execution:** Run tests locally using the bench command:
    ```bash
    bench run-parallel-tests --app press
    ```
    *Note: Ensure `allow_tests: true` is set in your `site_config.json`.*

## Frontend Tests (UI/E2E)

The Dashboard Vue app has an End-to-End (E2E) test suite using Playwright.

*   **Location:** `dashboard/tests-e2e/`
*   **Configuration:** `dashboard/playwright.config.ts`
*   **Execution:**
    1.  Ensure the Frappe backend is running (`bench start`).
    2.  Set up the required test users:
        ```bash
        bench --site test_site execute press.press.doctype.team.test_team.create_test_press_admin_team --kwargs '{"email": "test@example.com", "free_account": True, "skip_onboarding": True}'
        ```
    3.  Run Playwright tests from the `dashboard/` directory:
        ```bash
        npx playwright test
        ```
    4.  *(Optional)* Run tests with the browser visible:
        ```bash
        npx playwright test --headed
        ```

## Continuous Integration (CI)

Both test suites are run automatically in GitHub Actions on every Pull Request (defined in `.github/workflows/main.yaml`). Code coverage is collected and uploaded to Codecov.