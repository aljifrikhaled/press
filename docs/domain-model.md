# Domain Model

Press uses Frappe DocTypes to model its entire ecosystem. Here are the core entities.

## 1. Site (`press/press/doctype/site/`)
*   **Represents:** A single Frappe site, containing a specific database and associated files.
*   **Lifecycle:**
    *   **Creation:** Requested via the Dashboard, routed through `press.api.marketplace.create_site_for_app`, which creates the DocType and dispatches an `Agent Job` to provision it on the target bench.
    *   **Modification:** Users can add custom domains, update plans, or change configs. These trigger `Agent Job`s to sync configs to the server.
    *   **Archival/Deletion:** Archived sites are backed up and then deleted from the server, but the record is kept in Press for restoration.
*   **Relationships:** Belongs to one `Bench` and one `Server`. Owned by a `Team`.

## 2. Server (`press/press/doctype/server/`)
*   **Represents:** A compute instance (Virtual Machine or Bare Metal).
*   **Roles:** Can be an `App Server` (runs benches), `Database Server` (runs MariaDB), or `Proxy Server` (runs NGINX).
*   **Lifecycle:**
    *   **Creation:** Bootstrapped via cloud APIs (`Virtual Machine`), followed by Ansible playbook execution (`server.yml`) to install dependencies.
    *   **Management:** Press uses Ansible to configure users, mounts, swap, and Docker.
    *   **Monitoring:** The Agent continuously reports health. If down, the status changes to `Broken`.

## 3. Bench (`press/press/doctype/bench/`)
*   **Represents:** A Frappe Bench environment on a specific server.
*   **Purpose:** Groups Sites that run the exact same codebase (Frappe version and apps).
*   **Lifecycle:** Created via `Agent.new_bench()`. Automatically patched/updated when the associated `Release Group` is updated.

## 4. Release Group (`press/press/doctype/release_group/`)
*   **Represents:** A specific combination of apps (and branches) to be deployed.
*   **Usage:** Determines what codebase a `Bench` runs. When a Release Group is updated, all Benches attached to it pull the new code, affecting all Sites on those Benches.

## 5. Agent Job (`press/press/doctype/agent_job/`)
*   **Represents:** An asynchronous task sent to a managed server.
*   **State Machine:** `Undelivered` -> `Pending` (received by Agent) -> `Running` -> `Success` / `Failure`.
*   **Role:** Acts as the bridge between Press's intended state and the Agent's execution state. `hooks.py` runs `poll_pending_jobs` to continuously resolve these.

## 6. Team (`press/press/doctype/team/`)
*   **Represents:** A tenant (organization or user).
*   **Role:** The boundary for billing, subscriptions, and RBAC permissions. Users belong to Teams, and Teams own Sites and Servers.

## 7. Virtual Machine (`press/press/doctype/virtual_machine/`)
*   **Represents:** The cloud infrastructure abstraction layer.
*   **Role:** Maps to instances in AWS, Hetzner, Scaleway, or OCI. Contains methods like `provision_virtual_machine()` that interact directly with cloud provider APIs using libraries like `boto3`.
