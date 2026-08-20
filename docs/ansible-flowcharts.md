# Ansible Flowcharts

Press utilizes `ansible-runner` to programmatically execute playbooks directly from Python code. This usually happens immediately after a Virtual Machine finishes booting.

## Server Provisioning Flow (`server.yml`)

The primary playbook used to set up a brand new managed server is `press/playbooks/server.yml`.

```mermaid
graph TD
    Start([Virtual Machine Boots]) --> Trigger[Press Server.py: _setup_server]
    Trigger --> RunAnsible[Execute server.yml over SSH]

    subgraph Playbook: server.yml
        R1[Role: nat_iptables] --> R2[Role: essentials<br/>Installs curl, git, python]
        R2 --> R3[Role: user<br/>Creates 'frappe' user]
        R3 --> R4[Role: nginx<br/>Installs Reverse Proxy]
        R4 --> R5[Role: agent<br/>Installs Frappe Agent Daemon]
        R5 --> R6[Role: docker<br/>Installs Docker Engine]
        R6 --> R7[Role: Monitoring<br/>node_exporter, filebeat]
        R7 --> R8[Role: Security Hardening<br/>auditd, sshd_hardening]
    end

    RunAnsible --> Playbook
    R8 --> Success([Server Status: Active])
```

## Secondary Playbooks

Press also maintains specialized playbooks for different server types:

*   `self_hosted.yml`: Used when a user brings their own hardware. It bypasses cloud-specific volume mounting and network assumptions.
*   `update_agent.yml`: Used strictly to fetch the latest agent code from Git and restart the systemd service.
*   `mysql.yml`: Sets up MariaDB 10.6, tuning InnoDB buffers based on the server's RAM.