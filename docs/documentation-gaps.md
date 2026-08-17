# Documentation Gaps

This document identifies areas where the codebase lacks sufficient documentation or where current documentation requires expansion.

| Area | Existing Documentation | Actual Implementation | Gap | Priority |
| :--- | :--- | :--- | :--- | :--- |
| **Dashboard-Press Interface** | None | The Dashboard extensively uses `press.api.client.run_doc_method` to bypass standard REST patterns. | Needs a dedicated guide explaining how to trace `run_doc_method` calls to their Python implementation for new developers. | Critical |
| **Agent Payload Protocol** | None | Agent jobs (like `New Site` or `Backup Site`) construct complex JSON payloads (apps, configs, passwords). | There is no schema or documentation of the expected payload for each Agent Job type. | High |
| **Cloud Provider Mapping** | None | The `Virtual Machine` DocType abstracts AWS, Hetzner, OCI, and Scaleway. | Needs documentation on how cloud-specific metadata maps to standard Press server models. | Medium |
| **Ansible Flowcharts** | None | `server.yml` and `self_hosted.yml` install the Agent, setup NGINX, and harden security. | A visual flow of what happens during server provisioning is missing. | Medium |
| **Operational Troubleshooting** | None | Operations rely heavily on checking `Agent Job` error tracebacks and `Ansible Console Log`. | A playbook/guide for support engineers on debugging failed site creations or updates using Frappe Desk is needed. | High |
