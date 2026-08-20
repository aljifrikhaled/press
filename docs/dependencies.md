# Dependencies

Press relies on a combination of Python libraries for the backend and NPM packages for the frontend, alongside external system dependencies.

## Infrastructure & External Services
*   **MariaDB:** Database engine.
*   **Redis:** Caching and background job queuing (via RQ).
*   **Docker:** Used on managed servers to isolate Frappe Benches.
*   **Ansible:** (Python package `ansible`) Used by the Press Backend to configure managed servers.

## Backend Dependencies (Python)
Defined in `pyproject.toml`.

**Core Cloud & Automation:**
*   `ansible==3.4.0`: Executes playbooks to manage server state.
*   `boto3==1.39.14`: Interacts with AWS APIs (creating VMs, S3 backups).
*   `oci==2.180.0`, `hcloud==2.2.1`, `pydo==0.24.0`: SDKs for interacting with Oracle, Hetzner, and DigitalOcean cloud APIs.
*   `docker==6.1.2`: Docker SDK for Python.

**Integrations:**
*   `stripe`, `razorpay`: Payment gateway SDKs.
*   `python-telegram-bot`: Used for sending alerts to Telegram.
*   `PyGithub`: Used for OAuth integrations and checking custom app repositories.

**Metrics:**
*   `prometheus-client`, `prometheus-api-client`: Interacts with Prometheus for tracking server health and metrics.

## Frontend Dependencies (Vue/JS)
Defined in `dashboard/package.json`.

**Core:**
*   `vue` (v3): The core frontend framework.
*   `frappe-ui`: Frappe's official Vue component library, which provides the `frappeRequest` wrapper and UI components.
*   `tailwindcss`: CSS framework used for styling the Dashboard and Marketplace templates.
*   `vitepress`: Used for building this documentation site.

**Testing:**
*   `@playwright/test`: Used for End-to-End browser testing.