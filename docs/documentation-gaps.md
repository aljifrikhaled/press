# Documentation Gaps

This document previously identified areas where the codebase lacked sufficient documentation. **These gaps have now been addressed** in the following new documents:

| Previously Identified Gap | Addressed In | Description |
| :--- | :--- | :--- |
| **Dashboard-Press Interface** | `docs/dashboard-press-interface.md` | Dedicated guide explaining how to trace `run_doc_method` calls with a full example of the SaaS trial site creation workflow. |
| **Agent Payload Protocol** | `docs/agent-payload-protocol.md` | Examples of the JSON payload structures sent for creating and restoring sites via the Agent Job system. |
| **Cloud Provider Mapping** | `docs/cloud-provider-mapping.md` | Documentation on how the `Virtual Machine` DocType overrides client methods to abstract AWS, Hetzner, DigitalOcean, and OCI. |
| **Ansible Flowcharts** | `docs/ansible-flowcharts.md` | A Mermaid visual flowchart detailing the role execution sequence in the `server.yml` provisioning process. |
| **Operational Troubleshooting** | `docs/troubleshooting.md` | The troubleshooting guide was expanded into an operational playbook detailing how to trace errors down to the `Ansible Console Log` and `Agent Job Step` data fields. |
| **Installation Types** | `docs/installation-types.md` | Details the specific requirements and Ansible playbooks triggered for Unified, App, Database, and Proxy servers. |
| **Core Functions Guide** | `docs/implementation-guide.md` | Implementation guide covering `run_doc_method`, `Site Action` state machines, and background job handling. |
| **Partner Lifecycle** | `docs/partner-lifecycle.md` | Trace of the partner workflow from `Partner Leads` to automatic `Payout Order` generation and commission transfers. |
