# Agent Payload Protocol

The Frappe Agent daemon runs on the target servers and acts on instructions sent by the Press backend. Communication occurs via HTTP POST/DELETE requests.

## Constructing Agent Jobs

In Press, you don't call the Agent directly via HTTP inside business logic. You use the `Agent` class (`press/agent.py`) which acts as an abstraction layer.

When you call a method like `Agent(server).new_site()`, it creates an `Agent Job` DocType, which contains the **payload**.

### Example Payload: New Site Creation

When `Agent.new_site()` is invoked, it constructs the following payload structure and enqueues it.

**Endpoint:** `POST /benches/{bench_name}/sites`

**Payload (`data` field in Agent Job):**
```json
{
    "name": "customer-1.frappe.cloud",
    "apps": [
        "frappe",
        "erpnext",
        "hrms"
    ],
    "mariadb_root_password": "secure_db_password_from_press",
    "admin_password": "user_selected_admin_password",
    "config": {
        "encryption_key": "some_key",
        "frappe_user": "frappe"
    },
    "managed_database_config": {
        "database_host": "db.m6g.large.internal",
        "database_root_user": "root",
        "port": 3306
    }
}
```

### Example Payload: Restoring a Site

**Endpoint:** `POST /benches/{bench_name}/sites/{site_name}/restore`

**Payload:**
```json
{
    "apps": ["frappe", "erpnext"],
    "mariadb_root_password": "...",
    "admin_password": "...",
    "database": "https://s3.aws.../database.sql.gz",
    "public": "https://s3.aws.../public.tar.gz",
    "private": "https://s3.aws.../private.tar.gz",
    "skip_failing_patches": false
}
```

## Security

*   **Credentials:** Passwords (like DB roots or admin passwords) are passed in plain text via the JSON payload because the HTTP connection is secured via TLS (HTTPS) locally using ports `443` or `8443`.
*   **Verification:** Press polls the Agent for completion. The Agent logs stdout/stderr internally, and upon completion, sends the serialized execution trace back to Press, which is saved on the `Agent Job Step`.