# Deployment & Operations

## Deployment Architecture

In a production environment, Press relies on Frappe's standard deployment stack, augmented by its own Ansible orchestration.

1.  **Web Server (NGINX):** Acts as the reverse proxy, serving the compiled static assets from the Dashboard Vue app and routing `/api/` traffic to the Python backend.
2.  **Process Manager (Supervisor):** Manages the Gunicorn web workers and the Frappe RQ background workers.
3.  **Database:** MariaDB (typically version 10.6+).
4.  **Cache/Queue:** Redis.

## Build Process

Before deploying, static assets must be compiled. This is defined in `package.json`.

```bash
NODE_ENV=production yarn run build-all
```
This command compiles the Vue Dashboard using Vite, placing the output into `press/public/dashboard` and `press/www/dashboard.html`, which Frappe then serves. It also compiles Tailwind CSS for email templates and the marketplace.

## CI/CD Pipeline

The Continuous Integration pipeline is defined in `.github/workflows/main.yaml`.

*   **Triggers:** Push to branches or Pull Requests.
*   **Database Setup:** Spins up a MariaDB Docker container.
*   **Bench Setup:** Caches and installs `frappe-bench`.
*   **Testing:**
    *   **UI Tests:** Starts the bench, sets up test users (`create_test_press_admin_team`), and runs Playwright tests (`npx playwright test --project=chromium`).
    *   **Server Tests:** Runs Frappe Python tests using `bench run-parallel-tests --app press`.
*   **Coverage:** Uploads coverage reports to Codecov.

## Operations

### Log Management
*   **Frappe Logs:** standard `frappe-bench/logs/` (web, worker, scheduler).
*   **Agent Logs:** Agent interaction logs are written to `frappe-bench/logs/agent-jobs.json.log`.

### Troubleshooting Failed Operations
If a user operation (like creating a site) fails, the first step is to check the **Agent Job** DocType in the Frappe Desk.
*   Find the job by Site Name or Job Type.
*   Check the `Status` (Failure, Undelivered).
*   Check the `Data` JSON field, which often contains the exact traceback from the Agent running on the target server.
*   For Ansible provisioning failures, check the **Ansible Play** or **Ansible Console Log** DocTypes, which store the stdout from the `ansible-runner`.