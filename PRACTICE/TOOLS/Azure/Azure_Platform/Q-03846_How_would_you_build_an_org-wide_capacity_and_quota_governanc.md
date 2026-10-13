---
type: "practice"
difficulty: "Deep"
---

# How would you build an org-wide capacity and quota governance process on Azure so no team's spike starves another during a peak event, and quota requests don't become incident bottlenecks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Azure Messaging and Reliability](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/Azure_Messaging_and_Reliability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of building an org-wide capacity and quota governance process in Azure that ensures no team’s spike starves another during peak events and minimizes quota request bottlenecks, we need to focus on several key areas: resource allocation policies, automated monitoring, dynamic scaling mechanisms, and a robust governance framework. This solution will be both scalable and resilient, ensuring that critical services remain available even under high load.

The core idea is to leverage Azure's built-in tools for capacity planning and quota management combined with custom scripts and automation to enforce consistent practices across the organization. We'll start by defining clear service-level agreements (SLAs) and then use Azure policies and resource locks to ensure compliance. Automated alerts and reports will provide visibility into resource usage, allowing teams to proactively manage their workloads.

**Comparison:**

| Resource Type | Quota Limit | Current Usage | Alert Threshold (%) |
|---------------|------------|---------------|--------------------|
| vCPUs         | 100        | 75            | 80                 |
| Storage       | 1 TB       | 300 GB        | 90                 |

**Code:**

```python
# Example script for setting up Azure Policy via ARM template
{
    "if": {
        "allOf": [
            {"field": "type", "equals": "Microsoft.Storage/storageAccounts"},
            {"field": "Microsoft.Storage/storageAccounts/deleted", "exists": "false"}
        ]
    },
    "then": {
        "effect": "deny",
        "details": {
            "message": "Storage accounts are not allowed to be created.",
            "type": "Deployment",
            "role": "Contributor"
        }
    }
}
```

**Example input / output:**

```text
Input:
- Team A requests a temporary increase in vCPU for their application.
- The current quota limit is set to 100 vCPUs, and the team has already used 75.

Output:
- An automated alert triggers when the request exceeds 80% of the quota.
- A governance review process is initiated through Azure Policy to assess the request.
- If approved, a temporary increase in quota is granted, but a warning is sent to other teams about potential capacity impacts.
```

**Data:**

```json
{
  "storageAccounts": [
    { "name": "prod-db", "location": "eastus", "sku": { "name": "Standard_LRS" } },
    { "name": "test-logs", "location": "westus2", "sku": { "name": "Standard_GRS" } }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Resource Request
4. Is Quota Exceeded?
5. Governance Review
6. Quota Increase if Approved
7. Resource Allocation

