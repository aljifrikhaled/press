# Development Guide

## Running the Application Locally

Press requires two concurrent processes to run the full stack: the backend Frappe server and the frontend Vite development server.

1.  **Start Frappe Backend:**
    Open a terminal in your `frappe-bench` directory and start the Frappe processes:
    ```bash
    bench start
    ```
    This starts the Gunicorn web workers (on port 8000), Redis, and Python RQ background workers necessary for processing Agent Jobs.

2.  **Start Dashboard Frontend:**
    Open a second terminal, navigate to the Press app directory, and start the Vite dev server:
    ```bash
    cd apps/press/dashboard
    yarn dev
    ```
    Vite serves the UI (usually on port 8080) and automatically proxies API requests to the Frappe backend running on port 8000.

## Development Workflows

### Modifying Backend Logic
Backend logic is entirely contained within Frappe DocTypes (`press/press/doctype/`) and API routes (`press/api/`). After making Python changes, Frappe's Werkzeug development server automatically reloads.

### Modifying Frontend Code
Frontend code is located in `dashboard/src/`. The Vite dev server provides Hot Module Replacement (HMR). Changes to Vue components update in the browser instantly.

### Adding New Build Commands
The `package.json` in the root of the Press repo manages build commands.
*   `yarn build-app`: Builds the main dashboard Vue app.
*   `yarn build-email-css`, `yarn build-marketplace-css`: Builds Tailwind CSS files for specific static Frappe templates.
*   `yarn build-all`: Runs all build tasks for production.
