# System Overview

## What is Press?

Press is a managed Frappe hosting platform. It is the core software that powers [Frappe Cloud](https://frappecloud.com).

Press automates the orchestration of server infrastructure, app deployments (via Release Groups and Benches), multitenant database management, and site lifecycles. It abstracts away the complexities of server administration, allowing users to deploy and manage Frappe applications (like ERPNext) seamlessly.

## Who Uses Press?

*   **Tenants / Customers:** End users who need managed hosting for their Frappe applications without dealing with server administration. They interact primarily with the Dashboard.
*   **Partners:** Agencies and developers who resell hosting or build SaaS products on top of Frappe, utilizing features like Product Trials.
*   **System Administrators (Frappe Cloud Team):** Operators who manage the underlying infrastructure, monitor server health, and handle escalations using the Frappe Desk interface of the Press backend.

## Major Capabilities

*   **Site Management:** Create, archive, suspend, rename, and migrate Frappe sites.
*   **App Marketplace:** Install and manage Frappe apps on sites.
*   **Infrastructure Management:** Provision and scale virtual machines (App Servers, Database Servers, Proxy Servers) across various cloud providers (AWS, Hetzner, Scaleway, OCI).
*   **Backups:** Schedule, create, and restore logical and physical backups.
*   **Custom Domains & TLS:** Attach custom domains to sites and automatically provision TLS certificates.
*   **Billing & Subscriptions:** Manage usage limits, site plans, and billing for tenants and partners.
*   **Monitoring:** Monitor server health, MariaDB performance, and track slow queries or binlogs.

## Platform Structure

Press is not a single monolith but a distributed system comprising several distinct applications that work together:

1.  **Press Backend (Frappe App):** The control plane. It contains the core business logic, the domain model (Sites, Servers, Benches), billing, and exposes API endpoints. It runs on the Frappe framework.
2.  **Dashboard (Vue 3 SPA):** The frontend interface for tenants. It interacts exclusively with the Press backend via a REST API to manage resources.
3.  **Agent:** A Python application installed on every managed server. It acts as the execution plane, receiving HTTP requests from the Press Backend and running local Frappe/Bench commands or system tasks.
4.  **Ansible Playbooks:** Used by the Press Backend to provision and configure the underlying virtual machines (installing NGINX, Docker, monitoring agents, etc.).
5.  **Operational Tools (Libs):** A collection of system-level tools (e.g., `mariadb_monitor`, `mariadb_io_monitor`) deployed to servers to handle edge cases like Out-Of-Memory (OOM) recovery and database indexing.

## Typical Operation Lifecycle

When a user performs an action (e.g., creating a site):

1.  The user interacts with the **Dashboard**.
2.  The Dashboard makes an API request to the **Press Backend**.
3.  The Press Backend updates its database (Frappe DocTypes) to reflect the intended state and creates an **Agent Job**.
4.  A background worker (Frappe RQ) picks up the Agent Job and sends an HTTP request to the **Agent** on the target server.
5.  The **Agent** executes the task locally (e.g., running `bench new-site`).
6.  The **Press Backend** polls the Agent for the job's status. Once successful, the backend updates its state (marking the Site as "Active").
7.  The **Dashboard** reflects the updated state to the user.
