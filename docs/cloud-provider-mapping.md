# Cloud Provider Mapping

Press uses a generic abstraction layer to manage infrastructure across multiple cloud providers. This is handled centrally by the `Virtual Machine` DocType (`press/press/doctype/virtual_machine/virtual_machine.py`).

## Supported Providers
*   **AWS EC2** (via `boto3`)
*   **Hetzner** (via `hcloud`)
*   **Oracle Cloud (OCI)** (via `oci`)
*   **DigitalOcean** (via `pydo`)
*   **Frappe Compute** (Internal API)

## How Abstraction Works

When a user or a background process initiates the creation of a new server, a `Virtual Machine` document is created first.

The `Virtual Machine` DocType overrides the `.client()` method to return the correct SDK connection based on the selected `cloud_provider`:

```python
def client(self, client_type=None):
    cluster = frappe.get_doc("Cluster", self.cluster)
    if self.cloud_provider == "AWS EC2":
        return boto3.client("ec2", region_name=self.region, ...)
    if self.cloud_provider == "Hetzner":
        return HetznerClient(token=cluster.get_password("hetzner_api_token"))
    # ...
```

### Standardizing Infrastructure Actions

Regardless of the underlying provider, Press standardizes the following actions:
1.  **`provision_virtual_machine()`**: Calls the cloud API to boot a server using a pre-defined OS image and passes a Cloud-Init script (`cloud-init.yml.jinja2`).
2.  **`sync_virtual_machine()`**: Polls the cloud provider to fetch the Public/Private IPs, vCPUs, and RAM, updating the Frappe DocType.
3.  **`reboot_virtual_machine()`**: Issues a hard restart.
4.  **`destroy_virtual_machine()`**: Terminates the instance.

Once the Virtual Machine is marked as Active, Press creates the corresponding `Server` DocType and begins the Ansible provisioning phase.