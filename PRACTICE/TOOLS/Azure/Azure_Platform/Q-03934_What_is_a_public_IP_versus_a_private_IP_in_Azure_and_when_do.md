---
type: "practice"
difficulty: "Basic"
---

# What is a public IP versus a private IP in Azure and when do you assign a static public IP?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Networking and Global Traffic](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Networking_and_Global_Traffic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Azure, both public IPs and private IPs are essential components of networking, but they serve different purposes. A public IP is an external address that can be accessed from the internet directly, whereas a private IP is used within your virtual network to communicate with resources inside it. Static public IPs are assigned when you need consistent, globally reachable addresses for services like web servers or load balancers.

**Code:**

```python
# Example code to create a static public IP in Azure using the Azure SDK (Python)
from azure.identity import DefaultAzureCredential
from azure.mgmt.network import NetworkManagementClient

credential = DefaultAzureCredential()
subscription_id = "your_subscription_id"
network_client = NetworkManagementClient(credential, subscription_id)

resource_group_name = "myResourceGroup"
public_ip_name = "myPublicIP"

# Create a public IP with a static allocation
parameters = {
    'location': 'East US',
    'public_ip_allocation_method': 'Static'
}

network_client.public_ips.begin_create_or_update(resource_group_name, public_ip_name, parameters)
```

**Example input / output:**

```

To illustrate the creation of a static public IP in Azure using an ARM template:

JSON
Copy
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "publicIPName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/publicIpAddresses",
      "apiVersion": "2020-06-01",
      "name": "[parameters('publicIPName')]",
      "location": "East US",
      "properties": {
        "publicIpAllocationMethod": "Static"
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Create Static Public IP
6. Is the resource in a publicsubnet?
7. Assign the static public IP toan external-facing resource
8. Do you need globalreachability for this internalresource?
9. Consider using a NAT gatewayor Azure Front Door with aprivate IP
10. No action needed; use onlyprivate IPs within your VNet

