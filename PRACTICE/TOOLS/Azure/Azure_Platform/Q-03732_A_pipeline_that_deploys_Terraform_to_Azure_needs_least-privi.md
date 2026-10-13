---
type: "practice"
difficulty: "Deep"
---

# A pipeline that deploys Terraform to Azure needs least-privilege; how do you scope the service principal or federated identity so it can't touch other subscriptions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Pipelines and Release Approvals](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Pipelines_and_Release_Approvals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a pipeline deploying Terraform to Azure uses least-privilege principles, we need to carefully manage the permissions of the service principal or federated identity used by this deployment. The key idea is to limit its access strictly to the necessary resources and subscriptions while ensuring it can perform its intended operations effectively.

Firstly, let's clarify that we are dealing with a scenario where a pipeline needs to deploy Terraform configurations to Azure but must not have broader permissions than required. This involves understanding the principles of least privilege (LP) in the context of cloud deployments and how to implement them using Azure roles and identities.

**Comparison:**

```
Role	Actions	Scope

Contributor
	read, write, delete
	create, update
	list, start, stop, restart
		subscription / resource group
```

**Code:**

```python
# Example Python code for creating a service principal with constrained permissions
import azure.identity
from azure.mgmt.resource import ResourceManagementClient

subscription_id = "your-subscription-id"
tenant_id = "your-tenant-id"

client_id = "your-service-principal-client-id"
secret = "your-service-principal-secret"

credential = azure.identity.ClientSecretCredential(tenant_id, client_id, secret)
resource_client = ResourceManagementClient(credential, subscription_id)

# Create a resource group in the target subscription
resource_group_name = "target-resource-group"
location = "eastus"  # Choose an appropriate location

resource_client.resource_groups.create_or_update(
    resource_group_name,
    {
        "location": location,
    }
)
```

**Example input / output:**

```

For example, if we have two subscriptions (Subscription A for development and Subscription B for production), our pipeline should be able to deploy resources in Subscription B but not in Subscription A. We would assign the service principal a Contributor role only within Subscription B.

PLAINTEXT
Copy
Service Principal | Role Assignment | Target Resource Group/Subscription
---------------------------------------------------
| Contributor     | Subscription B  | Development (No access)
|                 |                |
| Contributor     | Contributor     | Production         (Full access)
```

**Data:**

```json
{
  "subscription_id": "your-subscription-id",
  "tenant_id": "your-tenant-id",
  "client_id": "your-service-principal-client-id",
  "secret": "your-service-principal-secret"
}

To achieve this, we need to understand the Azure roles and permissions model. Specifically, we will use a Contributor role with scoped access or create a custom role that limits the service principal's actions to Terraform-specific operations.
```

**Flow:**

1. No
2. Yes
3. Create Service Principal
4. Assign Roles?
5. Deploy to Subscription A (Noaccess)
6. Assign Contributor Role toSubscription B
7. Deploy to Subscription B (Fullaccess)

