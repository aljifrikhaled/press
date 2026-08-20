# How to Read This Documentation (Learning Paths)

Welcome to the Press documentation. Because Press is a complex distributed system covering frontend, backend, orchestration, and infrastructure, reading everything linearly might be overwhelming.

Instead, choose the learning path that best fits your role or immediate goal.

---

## 🧭 Path 1: The "10,000-Foot View" (For Everyone)
If you just joined the team or want to understand what the product does before diving into code.

1.  **[System Overview](system-overview.md):** Start here. It explains what Press is, who uses it, and the basic lifecycle of operations.
2.  **[Domain Model](domain-model.md):** Read this to understand the core nouns of the system (Sites, Servers, Benches, Release Groups, etc.).
3.  **[System Architecture](architecture.md):** Look at the Mermaid diagram to understand how the Dashboard, Press backend, Redis workers, Ansible, and the Frappe Agent connect.

---

## 💻 Path 2: New Backend Developer
If you need to fix a bug in the Python backend or add a new feature that the Dashboard will use.

1.  **[Installation & Setup](installation.md):** Follow these steps to get your local Frappe bench and Vite server running.
2.  **[Implementation Guide](implementation-guide.md):** Crucial read. Learn how `run_doc_method` works and how to dispatch background jobs safely.
3.  **[Dashboard-Press Interface](dashboard-press-interface.md):** Understand how the Vue frontend talks to the Python backend.
4.  **[Workflows](workflows.md):** Read the "Site Creation" workflow to see how asynchronous jobs flow through the system.
5.  **[Testing](testing.md):** Learn how to run the parallel tests to verify your changes.

---

## 🚀 Path 3: DevOps / Infrastructure Engineer
If your job is to manage the AWS/Cloud infrastructure, handle server scaling, or debug failed server provisions.

1.  **[Infrastructure Sizing (AWS)](infrastructure-sizing.md):** Understand how Press allocates memory and scales workers, and see the recommended instance types for 70 vs 3,000 sites.
2.  **[Installation Types](installation-types.md):** Understand the difference between an App Server, a Database Server, and a Proxy Server.
3.  **[Cloud Provider Mapping](cloud-provider-mapping.md):** Learn how Press interacts with AWS/Hetzner APIs to boot Virtual Machines.
4.  **[Ansible Flowcharts](ansible-flowcharts.md):** See exactly what roles are executed when a new server is provisioned via `server.yml`.
5.  **[Operational Troubleshooting](troubleshooting.md):** Your daily survival guide. Learn how to trace an `Agent Job` failure or an `Ansible Console Log` error.

---

## 🤖 Path 4: Agent & Orchestration Deep Dive
If you need to modify how Press talks to the managed servers or need to update the Agent daemon itself.

1.  **[Agent Protocol](agent-protocol.md):** Understand the polling mechanism, HTTP auth, and failure retries.
2.  **[Agent Payload Protocol](agent-payload-protocol.md):** Look at the exact JSON structures sent to the Agent for things like creating or restoring sites.
3.  **[Workflows](workflows.md):** Read the "Server Provisioning" and "Agent Update" workflows to see how the agent lifecycle is managed.

---

## 💼 Path 5: Product & Business Logic
If you are working on billing, partnerships, SaaS trials, or marketplace logic.

1.  **[Partner Lifecycle](partner-lifecycle.md):** Trace the path from a user becoming a partner, getting leads, calculating MRR, and receiving monthly Payout Orders.
2.  **[Dashboard-Press Interface](dashboard-press-interface.md):** See the example of how a "Product Trial Request" triggers backend site creation.
3.  **[Configuration](configuration.md):** Understand how pricing, Docker registries, and integrations are configured via Frappe DocTypes like `Press Settings`.
