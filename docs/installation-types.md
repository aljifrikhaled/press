# Installation Types

Press supports multiple deployment topologies for managed servers. The configuration and requirements differ slightly depending on the "Server Type" provisioned.

## Server Types

Press categorizes servers into three primary operational roles, plus a unified development role.

### 1. Unified Server (Development / Low-Cost)
A Unified Server runs the proxy, app bench, and database all on a single virtual machine. This is primarily used for small, non-critical setups or dedicated self-hosted servers.

*   **Requirements:** Minimum 2 vCPU, 4GB RAM (e.g., `c6i.large`).
*   **Installation Mechanism:** Press runs the `server.yml` or `unified_server.yml` Ansible playbook. This provisions Docker, NGINX, MariaDB 10.6, and the Frappe Agent.

### 2. App Server
An App server only runs the Frappe codebase (Python, Node.js, Gunicorn, Socket.io). It does *not* run a database.

*   **Requirements:** Typically compute and memory optimized (e.g., `m6i.xlarge` or `c6i.xlarge`).
*   **Installation Mechanism:** Press runs the `server.yml` playbook, skipping the database roles. It mounts persistent shared volumes via NFS (if using shared benches) and configures the Frappe Agent.

### 3. Database Server
A dedicated MariaDB host. Frappe sites are notoriously database-heavy, requiring significant InnoDB buffer pools.

*   **Requirements:** Memory optimized instances (e.g., `r6i.large` up to `r6i.4xlarge`).
*   **Installation Mechanism:** Press runs the `mysql.yml` playbook to configure MariaDB. It automatically tunes the `innodb_buffer_pool_size` based on the detected hardware RAM. It also installs the `mariadb_monitor` daemon for OOM protection.
*   **Note:** Instead of an EC2 DB Server, Press can optionally map to managed databases like AWS RDS.

### 4. Proxy Server
Handles incoming public traffic, SSL termination, and routes requests to the correct App Server.

*   **Requirements:** Compute optimized, low memory (e.g., `c6i.large`).
*   **Installation Mechanism:** Press configures NGINX via `proxy.yml`. It dynamically updates upstream configurations as sites migrate between App Servers.

---

## Server Provisioning Lifecycle

Whenever a new Server is added to Press (via the `Virtual Machine` abstraction or as a `Self Hosted Server`), Press executes the following standard flow:

1.  **Bootstrapping:** Cloud Init or manual SSH key insertion occurs.
2.  **Queueing Setup:** `press.press.doctype.server.server.setup_server` is called, putting the server in an `Installing` state.
3.  **Ansible Execution:** The `ansible-runner` subprocess triggers `press/playbooks/server.yml`.
4.  **Role Execution:** Essential packages, Docker, NGINX, Filebeat (for centralized logging), and the `Frappe Agent` are installed.
5.  **Agent Registration:** The Press backend generates a unique `agent_password` and uses it to authenticate future HTTP requests to the newly installed Agent daemon.