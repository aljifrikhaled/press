# Agent Protocol

The Frappe Agent is a separate daemon installed on managed servers. It listens for instructions from the Press Backend and executes system or Frappe Bench commands.

## Installation & Discovery

1.  **Installation:** When a Server is provisioned in Press, the `setup_server()` method runs an Ansible playbook (`server.yml`). This playbook connects via SSH, installs Docker, dependencies, and pulls/installs the Frappe Agent service.
2.  **Credentials:** During provisioning, Press generates an `agent_password` and stores it in the `Server` DocType.
3.  **Discovery:** Press knows the IP address of the Server. The Agent runs on port `443` (or `8443` for some configurations). Press connects directly to `https://{server_ip}:{port}`.

## Communication Flow

All communication is initiated by the **Press Backend** making HTTP requests to the Agent.

1.  **Job Dispatch:**
    *   Press creates an `Agent Job` DocType in the `Undelivered` state.
    *   A Frappe background worker runs `deliver_to_agent()` which makes an HTTP POST request to the Agent's API via `press/agent.py`.
    *   **Authentication:** Requests are authenticated using HTTP Basic Auth (Username: `root` or similar, Password: `agent_password`).
    *   **Payload:** Sent as JSON. Example payload for a New Site includes `apps`, `admin_password`, `mariadb_root_password`, etc.
    *   **Response:** If the Agent accepts the job, it returns an ID. Press updates the `Agent Job` to `Pending`.

2.  **Job Polling:**
    *   Because jobs (like installing a site or backing up) take time, they run asynchronously on the Agent.
    *   Press uses a scheduled job (`poll_pending_jobs` in `press/press/doctype/agent_job/agent_job.py`, scheduled every minute in `hooks.py`).
    *   Press sends an HTTP GET request to the Agent to fetch the status of all active job IDs for that server.
    *   The Agent responds with the job status (`Success`, `Failure`, `Running`).
    *   Press updates the `Agent Job` DocType.

3.  **Callback Processing:**
    *   Once a job reaches `Success` or `Failure`, `process_job_updates()` is called.
    *   This triggers specific handlers (e.g., `process_new_site_job_update()`) which update the system state (e.g., setting a Site to Active, or triggering a rollback).

## Failure Handling

*   **Retries:** If the Agent is temporarily unreachable (e.g., HTTP timeout), the job remains `Undelivered`. Press periodically retries undelivered jobs using exponential backoff (`retry_undelivered_jobs` in `agent_job.py`).
*   **Timeout:** Jobs have predefined timeouts (e.g., 4 hours). If a job doesn't finish, it may be marked as a Failure.
*   **Server Down:** Repeated connection failures cause the `Server` DocType status to change to `Broken`.

## Endpoints

Example endpoints exposed by the Agent (called by `press/agent.py`):
*   `POST /benches` - Create a new bench.
*   `POST /benches/{bench}/sites` - Create a new site.
*   `POST /benches/{bench}/restart` - Restart a bench.
*   `DELETE /benches/{bench}/sites/{site}/apps/{app}` - Uninstall an app.
