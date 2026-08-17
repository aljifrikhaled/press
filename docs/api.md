# API Interface

The Dashboard interacts with the Press Backend exclusively via HTTP REST APIs.

## The `frappeRequest` Wrapper

The Dashboard Vue app uses `frappeRequest()` (provided by `frappe-ui`) to communicate with the backend. This wrapper automatically attaches the `X-Press-Team` header (found in `dashboard/src/main.js`) to ensure requests are scoped to the currently active tenant/team.

## Core Interface: `run_doc_method`

Unlike standard REST APIs where every action has a dedicated endpoint, Press heavily utilizes Frappe's RPC capabilities. Most business logic is triggered by calling Python methods defined on DocTypes.

The primary endpoint for this is `press.api.client.run_doc_method`.

**Example: Creating a SaaS Trial Site**
*   **Trigger:** `dashboard/src/pages/signup/SetupSite.vue:132`
*   **Payload:**
    ```javascript
    frappeRequest({
        url: 'press.api.client.run_doc_method',
        args: {
            dt: 'Product Trial Request', // DocType
            dn: 'PTR-0001',              // Document Name
            method: 'create_site',       // Method on the Python class
            args: { subdomain: 'mysite', domain: 'frappe.cloud' }
        }
    })
    ```
*   **Backend execution:** This routes to `press/api/client.py` -> `run_doc_method()`, checks permissions, and executes `create_site()` on the specific `Product Trial Request` document in Python.

## Whitelisted Endpoints

Some complex logic that doesn't map cleanly to a single document method has dedicated API endpoints. These are Python functions decorated with `@frappe.whitelist()`.

**Notable Endpoints:**
*   **Marketplace / Installation:**
    *   `press.api.marketplace.create_site_for_app`: Determines if the app should go on a public or private bench, allocates resources, and triggers the Site DocType creation.
    *   `press.api.marketplace.options_for_quick_install`: Fetches valid candidate benches/servers for installing an app.
*   **Dashboard Metadata:**
    *   `press.api.dashboard.all`: Fetches summary counts of sites by status.
*   **GitHub Integration:**
    *   `press.api.github.*`: Handles OAuth flows and fetching repositories for custom app deployments.

## Authentication

*   **Tenants (Dashboard):** Authenticate using standard Frappe session cookies (`sid`). The Vue app checks `document.cookie.includes('user_id')` to verify login state.
*   **Agent to Press:** Typically, the Agent does not call *into* Press; Press polls the Agent. However, if the Agent needs to upload files (like backups), it uses standard Frappe API key/secret or pre-signed S3 URLs.
