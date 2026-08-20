# Infrastructure Sizing (AWS)

This document outlines the required infrastructure to deploy Press on AWS, based strictly on the internal allocation algorithms and logic found in the Press codebase.

Because Press is a distributed platform, it separates the **Control Plane** (managing the system) from the **Execution Plane** (hosting the customer sites).

*Note: For production workloads, this guide avoids burstable instance families (`t3`/`t4g`) as per standard practices, mapping closely to the `m6i`, `c6i`, and `r6i` families explicitly supported in Press's internal `Server Plan` fixtures and cloud provider mappings.*

---

## Understanding Press Sizing Logic

Press does **not** rely on hardcoded limits for "number of sites per server." Instead, it dynamically allocates resources and scales workers based on **workloads** and **usable RAM**:

1.  **Workload:** Press calculates the total `workload` of a Bench by summing the `cpu_time_per_day` of all sites on that Bench (`press/press/doctype/bench/bench.py`).
2.  **Usable RAM:** The server calculates its `usable_ram` as `max(ram - 3000, ram * 0.75)` to leave overhead for the OS disk cache and critical system processes (`press/press/doctype/server/server.py`).
3.  **Worker Allocation (`auto_scale_workers`):** Press allocates Gunicorn (web) workers and Background (RQ) workers to a bench based on its share of the server's total workload. It assumes each Gunicorn worker requires `150MB` (`GUNICORN_MEMORY`) and each background worker set requires `240MB` (`BACKGROUND_JOB_MEMORY`).
4.  **Capacity Incidents:** Press continuously monitors the memory map (`_refresh_bench_pool_and_raise_capacity_incidents`) across clusters. If a server has insufficient free memory (less than `300MB` available) for new benches, it raises a "Server Down: Insufficient bench capacity" incident.

---

## 1. Medium Scale Deployment: 70 Sites

To host 70 active customer sites, you need a highly available but compact setup. Assuming a balanced workload (mix of active and inactive sites):

### Architecture (4 Servers)

**1. Press Control Server**
This server runs the Press backend, the Dashboard UI, and the background workers that dispatch jobs to the agents.
*   **Role:** Control Plane, Background Jobs, Ansible Orchestration.
*   **AWS Instance:** `m6i.large` (2 vCPU, 8GB RAM). Ensures enough memory for Python RQ workers and `ansible-runner` subprocesses.
*   **Storage:** 50GB gp3 (For Press DB, logs, and playbooks)

**2. Proxy Server**
Press configures NGINX on this server to route incoming traffic to the correct internal App Server. It handles SSL termination.
*   **AWS Instance:** `c6i.large` (2 vCPU, 4GB RAM). Compute-optimized for handling SSL handshakes.
*   **Storage:** 30GB gp3

**3. Database Server**
Frappe applications are heavily database-dependent. Separating the database prevents poorly optimized queries on one site from crashing the web server.
*   **Role:** MariaDB 10.6 Host for all customer sites.
*   **AWS Instance:** `r6i.large` (2 vCPU, 16GB RAM) OR **AWS RDS for MariaDB**. Memory optimized for InnoDB buffer pools.
*   **Storage:** 200GB gp3 (Provisioned IOPS recommended if high traffic).

**4. App Server (Execution Plane)**
This server runs the actual Frappe Benches. Based on the `auto_scale_workers` logic, a 16GB instance provides `~12GB` of `usable_ram`.
*   **Role:** Frappe Agent, Frappe Benches.
*   **AWS Instance:** `m6i.xlarge` (4 vCPU, 16GB RAM).
*   **Storage:** 150GB gp3 (For site assets, public/private files).
*   **Scaling Note:** If the combined `cpu_time_per_day` of the 70 sites increases to the point where Gunicorn/RQ memory allocation hits the limit, Press will generate a Capacity Incident, prompting you to add another `m6i.xlarge` to the cluster.

---

## 2. Enterprise Scale Deployment: 3,000 Sites

Scaling to 3,000 sites requires expanding the execution plane. Press handles this automatically if servers are assigned to the correct Clusters.

### Architecture (30+ Servers)

**1. Control Plane Fleet**
The orchestration layer must be robust to handle thousands of background jobs (backups, let's encrypt renewals, agent pinging).
*   **Press Web/API:** 2x `c6i.xlarge` (Behind an AWS Application Load Balancer).
*   **Press Workers (RQ):** 2x `c6i.xlarge` (Dedicated specifically to running `frappe.enqueue` jobs and Ansible playbooks).
*   **Press Database:** AWS RDS MariaDB `m6i.xlarge` (Multi-AZ).
*   **Redis Cache/Queue:** AWS ElastiCache for Redis `cache.m6g.large`.

**2. Proxy Server Fleet**
*   **Proxy Servers:** 3x `c6i.xlarge` (Behind an AWS Network Load Balancer).
*   *Note: Press manages the NGINX upstream mapping automatically when sites are moved between App Servers.*

**3. App Server Fleet (Execution Plane)**
Based on Press's `usable_ram` algorithm, an `m6i.2xlarge` (32GB) provides `~24GB` of usable memory for workers.
*   **App Servers:** 20x to 30x `m6i.2xlarge` (8 vCPU, 32GB RAM).
*   **Storage:** 250GB gp3 per server.
*   *Strategy:* Press evaluates `bench_workloads` and dynamically scales workers (`gunicorn_workers` and `background_workers`) up to `MAX_GUNICORN_WORKERS` based on the sum of `cpu_time_per_day` across the sites on that server. Add servers to the cluster pool as capacity incidents arise.

**4. Database Fleet (Sharding)**
A single MariaDB instance cannot handle the concurrent connections for 3,000 active ERPNext sites. Press natively supports mapping sites to specific "Database Servers".
*   **Database Servers:** 5x `r6i.2xlarge` (8 vCPU, 64GB RAM) or equivalent AWS RDS instances.
*   **Distribution:** ~600 databases per DB server.
*   **Storage:** 1TB+ gp3/io2 per server.

**5. Centralized Caching (Redis)**
Use centralized Redis to manage Frappe caching and Socket.io for the fleet.
*   **Redis Fleet:** AWS ElastiCache for Redis (Cluster mode enabled).