# Configuration

Press relies on two primary configuration scopes: Frappe's `site_config.json` (for backend Python logic) and frontend environment variables (for the Vue Dashboard).

## Frontend Environment Variables (Vue)

These variables are defined in the build environment or a `.env` file within the `dashboard/` directory.

| Variable | Purpose | Used By | Required |
| :--- | :--- | :--- | :--- |
| `SENTRY_URL` | Sentry host URL for frontend error tracking. | `vite.config.ts` | No (Production) |
| `SENTRY_ORG` | Sentry Organization ID. | `vite.config.ts` | No (Production) |
| `SENTRY_PROJECT` | Sentry Project ID. | `vite.config.ts` | No (Production) |
| `SENTRY_AUTH_TOKEN`| Sentry token for uploading sourcemaps. | `vite.config.ts` | No (Production) |

*Note: The frontend also relies heavily on configuration injected dynamically from the backend during boot via `window` variables (e.g., `window.press_dashboard_sentry_dsn`, `window.press_frontend_posthog_project_id`). This data is provided by `press.www.dashboard.get_context_for_dev`.*

## Backend Configuration (`site_config.json`)

These configurations are set via `bench set-config <key> <value>` and reside in the Frappe site's `site_config.json`.

| Key | Purpose | Used By | Required |
| :--- | :--- | :--- | :--- |
| `ignore_csrf` | Disables CSRF validation. | Frappe Backend | No (Dev only) |
| `allow_tests` | Allows parallel test execution. | Github CI / Testing | No (Testing) |

## Application Settings (Frappe DocTypes)

Unlike many modern applications that use `.env` files for everything, Frappe applications store extensive configuration directly in the database.

**DocType: `Press Settings`**
This DocType (Single) holds the majority of operational configuration, including:
*   Docker Registry URLs and Credentials (used when provisioning Benches).
*   Agent Git Repository URL and Branch.
*   Log Server configuration.
*   Billing integrations.

**DocType: `Server`**
Stores per-server secrets:
*   `agent_password`: The Basic Auth password used by Press to talk to the Agent running on that server. Generated dynamically on server creation.

## Cloud Integrations

Cloud provider credentials (AWS, Hetzner, Scaleway, OCI) are managed via specific Frappe DocTypes or standard SDK environment variables depending on the execution context (e.g., standard Boto3 config for AWS).
