# Installation & Local Setup

This guide explains how to set up Press for local development.

## Prerequisites

Ensure you have the following installed on your system:
*   **Python:** >= 3.10
*   **Node.js:** 18
*   **Package Managers:** `pip`, `yarn`
*   **Database:** MariaDB 10.6
*   **Cache/Queue:** Redis Server
*   **Frappe Bench:** `pip install frappe-bench`

## Local Development Setup

Because Press is a Frappe application, it runs within a Frappe Bench environment.

1.  **Initialize a Bench:**
    ```bash
    bench init frappe-bench --frappe-branch version-15
    cd frappe-bench
    ```

2.  **Get Press App:**
    ```bash
    bench get-app https://github.com/frappe/press.git
    ```

3.  **Create a New Site:**
    ```bash
    bench new-site test_site
    ```
    *Note: Add `127.0.0.1 test_site` to your `/etc/hosts` file.*

4.  **Install Press on the Site:**
    ```bash
    bench --site test_site install-app press
    ```

5.  **Install Frontend Dependencies:**
    Navigate to the dashboard directory and install node modules.
    ```bash
    cd apps/press
    yarn install
    ```

6.  **Set up Pre-commit Hooks:**
    ```bash
    ./setup-pre-commit.sh
    ```

## Development Configuration

If you encounter `CSRFTokenError` during local development, disable CSRF checking in your site config.

```bash
bench --site test_site set-config ignore_csrf 1
```

To run Playwright UI tests, you also need test users. Press provides a helper script:
```bash
bench --site test_site execute press.press.doctype.team.test_team.create_test_press_admin_team --kwargs '{"email": "test@example.com", "free_account": True, "skip_onboarding": True}'
bench --site test_site set-password "test@example.com" "your_password"
```

## Running the Services

You need to run two separate processes for full development:

**1. Frappe Backend:**
From the `frappe-bench` directory, start the Frappe processes:
```bash
bench start
```

**2. Dashboard Frontend (Vite):**
From the `apps/press` directory, start the Vite development server:
```bash
yarn dev
```
The Vite server automatically proxies `/api`, `/app`, and `/files` requests to the running Frappe backend on port `8000`. Access the dashboard typically at `http://localhost:8080`.
