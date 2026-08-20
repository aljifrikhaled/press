# Operational Troubleshooting

This guide provides deep-dive steps for debugging common operational issues in Press. Since Press relies heavily on asynchronous queues and remote agents, troubleshooting usually involves tracing the state across multiple DocTypes.

## 1. Agent Job Failures (e.g., Site Creation Failed)

When a user triggers an action (like Site Creation or Backup), Press delegates it to the Agent. If it fails, the error lies on the target server but is synced back to Press.

**Step-by-Step Debugging:**
1.  **Locate the Agent Job:** Log into Frappe Desk and open the `Agent Job` list.
2.  **Identify the Job:** Filter by `Site` name or `Job Type` (e.g., `New Site`, `Backup Site`).
3.  **Investigate Status:**
    *   If status is `Undelivered`: Press cannot reach the Agent via HTTP. Check server network rules or Agent service status.
    *   If status is `Failure`: The Agent received the job, executed it, and the Frappe bench command crashed.
4.  **Read the Traceback (Crucial):** Open the specific `Agent Job Step` records linked to the Job. The `Data` field contains a JSON payload. Look for the `"traceback"` or `"output"` keys. This contains the exact `stderr` from the remote server (e.g., standard `bench` error traces indicating a missing app or a bad password).

## 2. Server Provisioning Failures (Ansible)

When a Server is created, Press uses `ansible-runner` to set it up. If a server gets stuck in `Installing` or `Broken`:

**Step-by-Step Debugging:**
1.  **Check Virtual Machine Status:** Ensure the underlying `Virtual Machine` DocType status is `Active` and has an IP address. If it doesn't, the failure occurred at the Cloud Provider API level (check Frappe Error Logs).
2.  **Locate Ansible Logs:** If the VM is active, the failure happened during configuration. Open the `Ansible Console Log` DocType.
3.  **Read the Output:** The console log stores the raw output of the playbook (`server.yml`). Look for red "FAILED" lines.
    *   *Common Issue:* `unreachable` - Press could not SSH into the VM. Ensure the Cloud-Init script injected the correct SSH keys.
    *   *Common Issue:* Apt/Yum lock errors - Wait and retry.

## 3. Storage and Capacity Incidents

Press auto-scales workers and monitors disk space. If a server runs out of capacity:

1.  **Check Incidents:** Look at the `Incident` DocType. Press automatically creates incidents like "Insufficient bench capacity" when a cluster's usable RAM drops below thresholds.
2.  **Review Usable RAM:** Check the `Server` DocType. Ensure `usable_ram` is calculating correctly and that `workload` (sum of site CPU usage) hasn't spiked unnaturally.
3.  **Disk Expansion:** If disk space is low, Press attempts to use the cloud provider API to resize the EBS/Block volume automatically (via `extend_ec2_volume`). Check the `Add On Storage Log` to see if the auto-expansion failed.

## 4. Local Development: CSRF Errors

When developing the Dashboard locally against a local Frappe backend, you might encounter `CSRFTokenError` in the browser console.

**Fix:** Run this command to disable CSRF checks in your development bench:
```bash
bench --site test_site set-config ignore_csrf 1
```