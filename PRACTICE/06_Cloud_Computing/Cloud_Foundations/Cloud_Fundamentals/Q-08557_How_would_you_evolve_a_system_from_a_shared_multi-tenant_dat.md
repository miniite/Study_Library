---
type: "practice"
difficulty: "Expert"
---

# How would you evolve a system from a shared multi-tenant database to a per-tenant database as your biggest customers grow, without downtime for the small tenants?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The key to evolving a system from a shared multi-tenant database to a per-tenant database is through a gradual and controlled migration process that minimizes disruption. This involves creating isolated databases for each tenant while maintaining a seamless experience for the small tenants, using techniques like load balancers and stateless API gateways.

The core challenge here lies in ensuring that the transition does not cause downtime or degradation of service for smaller tenants, which might be dependent on shared resources. The solution involves carefully planning the migration process, leveraging cloud-native services, and implementing strategies to manage tenant isolation and resource allocation effectively.

#### Triage
This question is essentially asking how to handle a scale-up scenario where certain customers require more isolated resources while maintaining service continuity for all tenants. The interviewer is probing your understanding of database management, load balancing, statelessness, and the overall architecture and scaling considerations in cloud environments.

Step-by-Step Solution:

Assess Current Architecture: Evaluate the current multi-tenant setup to understand the dependencies, shared resources, and critical operations.
Identify Key Challenges: Determine how sharing affects performance, security, and data consistency for both large and small tenants.
Plan Migration Strategy:
Phase 1: Preparation – Create a new environment in your cloud provider that mirrors the current setup but with isolated databases.
Phase 2: Data Partitioning – Gradually migrate smaller tenant data to the new isolated databases, ensuring minimal impact on service availability.
Phase 3: Parallel Operation – Run both the shared and isolated environments side by side for a period to validate operations and identify any issues.
Phase 4: Cut Over – Once validated, perform a final migration of all tenants to the new architecture with zero downtime for small tenants.
Implement Load Balancers: Use cloud load balancers to distribute traffic evenly across both shared and isolated databases.
Ensure Statelessness: Design stateless API gateways that handle requests without dependency on tenant-specific state, ensuring horizontal scalability.
Resource Management: Implement auto-scaling policies for the new per-tenant databases based on real-time metrics.

#### Closing Summary
By carefully planning and executing a multi-phase migration strategy, we can ensure that larger tenants are not penalized while minimizing disruption to smaller tenants. This approach leverages cloud-native services like load balancers and stateless gateways to maintain service continuity during the transition period.

#### Common Mistakes/Follow-Ups

Failing to properly assess and plan for the dependencies within the current architecture.
Overlooking the need for gradual migration of data, which could lead to downtime or performance issues.
Not implementing statelessness in API design, leading to unexpected behavior when scaling out tenants.
Ignoring resource management strategies that can prevent over-provisioning and under-utilization.

**Comparison:**

| Tenant ID | Initial Database | Post-Migration Database |
|-----------|------------------|------------------------|
| T001      | sharedDB         | tenant-T001            |
| T002      | sharedDB         | tenant-T002            |

**Code:**

```python
# Example: Load Balancer Configuration Using Cloudflare's API
import requests

def update_load_balancer(lb_id):
    url = f"https://api.cloudflare.com/client/v4/loadbalancers/{lb_id}"
    headers = {
        "Authorization": "Bearer YOUR_API_TOKEN",
        "Content-Type": "application/json"
    }
    data = {
        "settings": {
            "algorithm": "ROUND_ROBIN",  # Change this based on load
            "pools": [
                {"name": "SharedDBPool", "weight": 50},
                {"name": "IsolatedDBPool", "weight": 50}
            ]
        }
    }
    response = requests.put(url, headers=headers, json=data)
    if response.status_code == 200:
        print("Load balancer updated successfully.")
    else:
        print(f"Failed to update load balancer: {response.json()}")

# Example Usage
update_load_balancer('LB-123456')
```

**Example input / output:**

```

Example Dry-Run:

Initial Setup: Shared database sharedDB with tenants T001 and T002.
Post-Migration Setup: Separate databases for tenants, e.g., tenant-T001 and tenant-T002.

Request/Response Example:

PLAINTEXT
Copy
Request:
GET /data?tenantId=T001

Response (Before Migration):
{
  "database": "sharedDB",
  "data": {...}
}

Response (After Migration):
{
  "database": "tenant-T001",
  "data": {...}
}
```

**Data:**

```json
{
  "tenantData": [
    {
      "id": "T001",
      "size": "small"
    },
    {
      "id": "T002",
      "size": "large"
    }
  ],
  "databaseConfiguration": {
    "sharedDatabaseName": "sharedDB",
    "isolatedDatabases": [
      { "tenantId": "T001", "dbName": "tenant-T001" },
      { "tenantId": "T002", "dbName": "tenant-T002" }
    ]
  }
}
```

**Flow:**

1. Evaluate Impact
2. Initial Setup
3. Tenant Growth
4. Assess Current Architecture
5. Identify Key Challenges
6. Plan Migration Strategy
7. Phase 1: Preparation
8. Phase 2: Data Partitioning
9. Phase 3: Parallel Operation
10. Phase 4: Cut Over
11. Implement Load Balancers
12. Ensure Statelessness
13. Resource Management

