# Dashboard-Press Interface

The Dashboard is a Vue 3 Single Page Application (SPA) that manages Frappe infrastructure on behalf of tenants. It does not use traditional REST methodologies (like `GET /sites`, `POST /sites`); instead, it heavily utilizes Frappe's RPC capabilities via `frappeRequest`.

## Understanding `run_doc_method`

The core function used to trigger backend workflows is `press.api.client.run_doc_method`. This function executes Python methods defined directly on Frappe DocType controllers.

### Example: Processing a SaaS Trial

When a user signs up for a trial via the Dashboard, the frontend needs to trigger the `create_site` method located in `press/saas/doctype/product_trial_request/product_trial_request.py`.

**Frontend Implementation (`dashboard/src/pages/signup/SetupSite.vue`):**
```javascript
frappeRequest({
    url: 'press.api.client.run_doc_method',
    args: {
        dt: 'Product Trial Request', // The DocType Name
        dn: 'PTR-0001',              // The Document Name (ID)
        method: 'create_site',       // The Python method to execute
        args: {
            subdomain: 'mysite',
            domain: 'frappe.cloud'
        },
    },
})
```

**Backend Execution (`press/api/client.py` -> `run_doc_method`):**
1. The API validates the currently logged-in user's permissions via `X-Press-Team` headers.
2. It verifies that the `method` is explicitly permitted to be executed by the Dashboard.
3. It loads the `Product Trial Request` document and calls `doc.create_site(subdomain='mysite', domain='frappe.cloud')`.

## Troubleshooting API Calls

If you are developing the Dashboard and an API call fails:
1.  **Check Network Tab:** Verify the `dt` (DocType) and `method` being sent.
2.  **Locate Python Code:** Look for the python file corresponding to the DocType (e.g., `press/press/doctype/site/site.py`).
3.  **Check Whitelisting:** Ensure the python method is accessible or that there are no permission blocks preventing the execution in `client.py`.