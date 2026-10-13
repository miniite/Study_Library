---
type: "practice"
difficulty: "Deep"
---

# A disk-attached VM in a scale set is losing data on scale-in; how do you separate compute from state and handle ephemeral vs managed disks correctly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Functions and Cosmos DB Tuning](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where an Azure Virtual Machine (VM) within a scale set is experiencing data loss during scale-in operations, the core issue lies in how compute resources are separated from stateful data, especially considering ephemeral vs managed disks. The key to resolving this lies in understanding the principles of state management and disk types in Azure.

Firstly, let's clarify what’s at play here:

Ephemeral Disks: These are temporary storage volumes that are deleted when a VM is deallocated or stopped. They are ideal for storing transient data.
Managed Disks: These persist across VM lifecycle events (like scale-in) and are essential for stateful data.

In this scenario, we need to ensure that the state of our application is stored on managed disks, while ephemeral disks can be used for temporary or non-critical data. Here’s a step-by-step solution from first principles:

Identify Stateful Data: Determine which components of your application require persistent storage and must survive scale-in operations.
Use Managed Disks: Assign managed disks to the VMs in the scale set that store stateful data. This ensures that when a VM is scaled out or in, its state remains intact.
Ephemeral Disk Usage: Use ephemeral disks for temporary storage needs, such as logs or cache. Ensure these are not critical to your application’s operation and can be recreated without issue.

Here’s a more detailed breakdown:

Step 1: Assess Application Needs

#### Cons
ider the nature of the stateful data (e.g., database, configuration files).
Determine which components require persistence across scale operations.

Step 2: Configure Managed Disks

Create managed disks and attach them to your VMs in the scale set.
Ensure these disks are properly configured for high availability and durability.

Step 3: Migrate Stateful Data

If you already have stateful data on ephemeral disks, plan a migration strategy. This might involve:
Backing up existing data.
Transitioning the data to managed disks.
Testing the transition thoroughly.

Step 4: Use Ephemeral Disks Wisely

Configure ephemeral disks for non-critical tasks like caching or logging.
Ensure these are not critical paths in your application’s operation and can be safely recreated.

Step 5: Scale Set Configuration

Review the scale set configuration to ensure it properly handles stateful data without losing it during scale-in operations.
Use appropriate auto-scaling policies that consider the health of managed disks.

**Comparison:**

```
VM State	Disk Type	Data Persistence
Running	Managed	Yes
Stopped	Managed	Yes
Running	Ephemeral	No
```

**Code:**

```python
# Example Python script to automate disk attachment/detachment in Azure

from azure.mgmt.compute import ComputeManagementClient
from azure.common.client_factory import get_client_from_auth_file

def attach_disk_to_vm(compute_client, resource_group_name, vm_name, managed_disk_id):
    try:
        compute_client.virtual_machines.create_or_update(
            resource_group_name,
            vm_name,
            {
                "storage_profile": {
                    "osDisk": {
                        "managed_disk": {"id": managed_disk_id},
                        "caching": "ReadWrite"
                    }
                }
            }
        )
        print(f"Managed disk {managed_disk_id} attached to VM {vm_name}")
    except Exception as e:
        print(f"Error attaching disk: {e}")

# Example usage
compute_client = get_client_from_auth_file(ComputeManagementClient)
attach_disk_to_vm(compute_client, "my-resource-group", "my-vm", "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/disks/my-managed-disk")
```

**Example input / output:**

```text
Input:
resource_group_name = "my-resource-group"
vm_name = "my-vm"
managed_disk_id = "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/disks/my-managed-disk"

Output:
Managed disk /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/disks/my-managed-disk attached to VM my-vm
```

**Data:**

```json
{
  "resource_group_name": "my-resource-group",
  "vm_name": "my-vm",
  "managed_disk_id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Compute/disks/my-managed-disk"
}
```

**Flow:**

1. Scale In
2. VM Deallocated
3. Managed Disk Detached
4. Stateful Data Lost
5. Ephemeral Disk Detached
6. Stateful Data Persisted

