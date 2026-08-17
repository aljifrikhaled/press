# Troubleshooting

This guide provides steps for debugging common operational issues in Press.

## 1. Job Failures (e.g., Site Creation Failed)

If a user reports that a Site Creation, Backup, or App Installation failed:
1.  **Locate the Agent Job:** Log into the Press backend (Frappe Desk) and go to the `Agent Job` list.
2.  **Filter:** Filter by the `Site` name or `Job Type` (e.g., `New Site`).
3.  **Check Status:** The status will likely be `Failure`.
4.  **Read the Traceback:** Look at the `Data` field (JSON format) on the Agent Job. The Agent returns the exact stdout/stderr from the Frappe Bench command that failed on the target server.

## 2. Server Provisioning Failures

If a new server gets stuck in the `Installing` state or changes to `Broken`:
1.  **Check Cloud Provider:** First, verify in the Cloud Provider console (AWS, Hetzner) that the Virtual Machine actually booted and passed health checks.
2.  **Check Ansible Logs:** Go to the `Ansible Play` or `Ansible Console Log` DocType in Frappe Desk. Look for the most recent play associated with the Server.
3.  **Review Output:** The log will contain the raw output of the `ansible-runner`. Identify which specific Ansible task failed (e.g., failed to connect via SSH, failed to install Docker).

## 3. Agent Connectivity Issues

If a Server status is `Broken` but the VM is running:
1.  **Verify Network:** Ensure the Press backend IP is whitelisted in the target server's firewall (port 443 or 8443).
2.  **Check Undelivered Jobs:** Go to `Agent Job` list and filter by `Status: Undelivered`. If jobs are piling up for a specific server, it means Press cannot reach the Agent via HTTP.
3.  **Check Agent Service:** SSH into the target server and check the agent service status: `systemctl status agent.service` (or equivalent).
4.  **Restart Agent:** If the agent is stuck, restart it on the target server. Press will automatically retry `Undelivered` jobs via the `retry_undelivered_jobs` background task.

## 4. CSRF Errors Locally

When developing locally, you may encounter `CSRFTokenError` in the browser console.
**Fix:** Run `bench --site test_site set-config ignore_csrf 1` to disable CSRF checks during local development.