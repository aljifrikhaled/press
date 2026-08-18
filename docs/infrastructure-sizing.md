# Infrastructure Sizing & AWS Deployment Guide

This document outlines the required infrastructure to deploy Press on AWS. Because Press is a distributed platform, it separates the **Control Plane** (managing the system) from the **Execution Plane** (hosting the customer sites).

*Note: For production workloads, this guide strictly avoids AWS burstable instance families (like `t3` or `t4g`) to ensure consistent CPU performance, relying instead on compute-optimized (`c6i`), general-purpose (`m6i`), or memory-optimized (`r6i`) families.*

---

## 1. Small Scale Deployment: 70 Sites

To host 70 active customer sites, you need a highly available but compact setup.

### Recommended Architecture (4 Servers)

**1. Press Control Server**
This server runs the Press backend, the Dashboard UI, and the background workers that dispatch jobs to the agents. *No customer sites run here.*
*   **Role:** Control Plane, Background Jobs, Ansible Orchestration.
*   **AWS Instance:** `m6i.large` (2 vCPU, 8GB RAM)
*   **Storage:** 50GB gp3 (For Press DB, logs, and playbooks)

**2. Proxy Server**
Press uses this server to route incoming traffic (e.g., `site1.frappe.cloud`) to the correct internal App Server. It handles SSL termination.
*   **Role:** NGINX Reverse Proxy, Let's Encrypt SSL.
*   **AWS Instance:** `c6i.large` (2 vCPU, 4GB RAM)
*   **Storage:** 30GB gp3

**3. Database Server**
Frappe applications are heavily database-dependent. Separating the database prevents poorly optimized queries on one site from crashing the web server.
*   **Role:** MariaDB 10.6 Host for 70 sites.
*   **AWS Instance:** `r6i.large` (2 vCPU, 16GB RAM) OR **AWS RDS for MariaDB**.
*   **Storage:** 200GB gp3 (Provisioned IOPS recommended if high traffic).

**4. App Server**
This server runs the actual Frappe Benches (Python processes, Gunicorn, Node/Socket.io) for the customer sites.
*   **Role:** Frappe Agent, Frappe Benches.
*   **AWS Instance:** `m6i.xlarge` (4 vCPU, 16GB RAM).
*   **Storage:** 150GB gp3 (For site assets, public/private files, and Docker images).
*   **Note:** 70 standard ERPNext sites can be comfortably packed onto a single 16GB RAM server. If traffic grows, Press allows you to provision a second `m6i.xlarge` and seamlessly migrate sites over.

---

## 2. Enterprise Scale Deployment: 3,000 Sites

Scaling to 3,000 sites requires moving from single servers to fleets, implementing database sharding, and utilizing dedicated caching.

### Recommended Architecture (30+ Servers)

At this scale, you must group sites into "Clusters" or use Press's ability to map Release Groups to specific server pools.

**1. Control Plane Fleet**
The orchestration layer must be robust to handle thousands of background jobs (backups, let's encrypt renewals, agent pinging).
*   **Press Web/API:** 2x `c6i.xlarge` (Behind an AWS Application Load Balancer).
*   **Press Workers (RQ):** 2x `c6i.xlarge` (Dedicated specifically to running `frappe.enqueue` jobs and Ansible playbooks).
*   **Press Database:** AWS RDS MariaDB `m6i.xlarge` (Multi-AZ).
*   **Redis Cache/Queue:** AWS ElastiCache for Redis `cache.m6g.large`.

**2. Proxy Server Fleet**
A single NGINX proxy will struggle with the SSL handshakes and connection tracking for 3,000 domains.
*   **Proxy Servers:** 3x `c6i.xlarge` (Behind an AWS Network Load Balancer).
*   *Note: Press manages the NGINX upstream mapping automatically when sites are moved between App Servers.*

**3. App Server Fleet (Execution Plane)**
Assuming a packing density of roughly 100-150 sites per server (depending on usage).
*   **App Servers:** 20x to 30x `m6i.2xlarge` (8 vCPU, 32GB RAM).
*   **Storage:** 250GB gp3 per server.
*   *Strategy:* Press allows you to designate servers for specific "Release Groups" (e.g., Servers 1-10 run Frappe v14, Servers 11-20 run Frappe v15).

**4. Database Fleet (Sharding)**
A single MariaDB instance cannot handle the concurrent connections and buffer pool requirements for 3,000 active ERPNext sites. You must "shard" the databases. Press natively supports assigning sites to different "Database Servers".
*   **Database Servers:** 5x `r6i.2xlarge` (8 vCPU, 64GB RAM) or equivalent AWS RDS instances.
*   **Distribution:** ~600 databases per DB server.
*   **Storage:** 1TB+ gp3/io2 per server.

**5. Centralized Caching (Redis)**
Instead of running Redis on every App Server, use centralized Redis to manage Frappe caching and Socket.io.
*   **Redis Fleet:** AWS ElastiCache for Redis (Cluster mode enabled).

---

## Required AWS Services Integration

Regardless of scale, Press relies on the following AWS services for a smooth deployment:

1.  **Amazon S3:** Required for offsite storage. Press will automatically upload daily logical (SQL) and physical (files) backups here.
2.  **AWS IAM (Identity and Access Management):** Press requires an IAM User with programmatic access (API Keys) to dynamically provision new VMs, resize EBS volumes, and manage snapshots. This interacts directly with the `Virtual Machine` and `Virtual Machine Volume` doctypes in Press.
3.  **Amazon Route 53:** For DNS automation. When users create sites, Press can automatically create the A/CNAME records if a wildcard domain is not sufficient.