---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-off between per-service databases (team autonomy) and a shared database (cheaper) purely on cost. When would the autonomy be worth the extra spend?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the trade-off between using per-service databases to achieve team autonomy versus employing a shared database for cost savings, we need to analyze both scenarios in terms of their financial implications and organizational benefits. The key elements to consider are infrastructure costs, data management complexity, operational overhead, and long-term strategic alignment.

#### Triage
Understanding the Question

The question is probing our understanding of how different database architectures impact cloud cost governance and team autonomy. We need to evaluate whether the benefits of autonomy (like faster development cycles or better data isolation) justify the increased expenses associated with separate databases compared to a shared one, which can be cheaper due to economies of scale.

#### First-Principles Solution
Per-Service Databases: Team Autonomy and Cost Implications
Infrastructure Costs: Each team gets its own database, leading to higher infrastructure costs as each service needs its own compute, storage, network, and backup resources.
Data Management Complexity: Separate databases increase the complexity of data management and synchronization across services. This can lead to additional overhead in maintaining consistent data models and preventing data inconsistencies.
Operational Overhead: Managing multiple databases requires more DevOps support for deployment, monitoring, security, and disaster recovery.
Scalability and Resilience: Each service may have different scaling needs, leading to potential over-provisioning or under-utilization of resources.
Shared Database: Cost Savings and Risks
Cost Efficiency: A shared database can leverage economies of scale, reducing costs per unit of storage and compute.
Operational Simplicity: Simplified data management processes with a single point of entry for updates and backups.
Resource Optimization: Centralized monitoring and security policies can optimize resource usage more effectively.

#### Closing Summary

The trade-off between per-service databases (team autonomy) and a shared database (cost efficiency) hinges on balancing development velocity, operational complexity, and financial constraints. While per-service databases offer teams the freedom to innovate quickly with tailored data management practices, they come at a higher cost due to infrastructure duplication. Conversely, a shared database simplifies operations but may hinder some teams' ability to scale or control their data environments independently.

#### Common Mistakes/Follow-Ups
Ignoring Data Consistency: Failing to enforce data consistency across multiple databases can lead to operational inefficiencies and data integrity issues.
Underestimating Long-Term Costs: Over-provisioning in per-service databases can result in unnecessary costs, while shared database inefficiencies might lead to underutilized resources.
Lack of Scalability Planning: Assuming that a single shared database will meet all future growth needs without proper planning can lead to bottlenecks.

**Code:**

```python
# Example cost model for per-service vs shared databases (simplified)
def calculate_cost(databases, service_count):
    # Per-service costs
    per_service_cost = 100 + 50 * service_count  # Base + storage cost per service

    # Shared database costs
    shared_database_cost = 2000 + 50 * databases  # Base + storage cost for shared db

    return {"per_service": per_service_cost, "shared_db": shared_database_cost}

service_count = 10
costs = calculate_cost(1, service_count)
print(f"Per-service cost: ${costs['per_service']}, Shared DB cost: ${costs['shared_db']}")
```

**Example input / output:**

```text
Input:
- Service Count: 10

Output:
Per-service cost: $600, Shared DB cost: $2500
```

**Data:**

```json
{
  "database": {
    "per_service": {
      "count": 10,
      "cost_per_unit": {"base": 100, "storage": 50}
    },
    "shared_db": {
      "count": 1,
      "cost_per_unit": {"base": 2000, "storage": 50}
    }
  },
  "services": [
    {"name": "Service A", "costs": {"per_service": 600}},
    {"name": "Service B", "costs": {"per_service": 600}}
  ]
}
```

