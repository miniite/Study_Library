---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide between putting a workload in a single larger AZ-resilient managed service versus splitting it across three smaller instances yourself, given a fixed reliability target and a tight budget?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Managed Services vs Self-hosting](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to put a workload in a single larger, AZ-resilient managed service or split it across three smaller self-managed instances within your budget constraints, you need to balance reliability, cost, and operational complexity. The key is understanding the trade-offs between using managed services (often more reliable due to built-in redundancy) versus managing infrastructure yourself with potentially lower costs but higher operational overhead.

TL;DR: Assess the reliability requirements, compare managed service vs. self-managed options based on cost, performance, and maintenance, then choose the option that best meets your budget while not compromising on reliability.

First, let’s triage the question:

What is being probed: The interviewer wants to evaluate how you balance reliability, cost, and operational complexity in cloud design.

#### Assumptions
Fixed reliability target: You need to meet a specific SLA.
Tight budget: Cost is a significant constraint.

Now let’s break this down from first principles:

Reliability Considerations:

Managed Service: Typically offers built-in redundancy and disaster recovery, often with higher uptime guarantees due to managed backups, patching, and monitoring.
Self-Managed Instances: Require more operational effort but can be tailored precisely to your needs.

Cost Analysis:

Managed Service Costs: Often have a predictable cost model, though there might be hidden costs like data egress or additional features.
Self-Managed Costs: Include instance costs, storage, networking, and any additional tools for monitoring and management. You may also need to budget for ongoing maintenance and support.

Operational Complexity:

Managed Service: Typically requires less operational effort as the provider handles most of the operations.
Self-Managed Instances: Require more hands-on management for setup, configuration, updates, and monitoring.

Reliability vs. Cost Trade-offs:

Evaluate if a single larger managed service can meet your reliability target within budget constraints.
If not meeting the target, consider whether splitting workloads across three smaller instances (self-managed or managed) provides better reliability at an acceptable cost increase.

Scalability and Performance Considerations:

Analyze how each option scales under load. Managed services often offer auto-scaling features.
Ensure that the self-managed approach can handle the expected traffic patterns without compromising performance.

Operational Risk:

Evaluate the risk of operational errors with both options, especially in terms of downtime and recovery time objectives (RTO).

#### Example Scenario

Suppose your reliability target is 99.95% uptime, and you have a tight budget of $100/month.

Managed Service: A managed database or application service might cost around $80/month with an SLA guaranteeing 99.95% uptime.
Self-Managed Instances: Three small instances (each costing about $20/month) could be used to achieve the same reliability, but you’d need to budget extra for monitoring tools, additional storage, and potential maintenance costs.

In this case, a single larger managed service would meet your needs within budget and reduce operational complexity significantly.

#### Closing Summary

To decide between these options, evaluate the cost, reliability, and operational overhead. A single larger, AZ-resilient managed service is often preferable for simplicity and predictability in costs and uptime guarantees. However, if you need to optimize every dollar or have specific requirements that can’t be met by managed services, splitting workloads across smaller self-managed instances might be necessary.

#### Common Mistakes/Follow-ups
Overestimating the reliability of self-managed systems.
Underestimating the hidden costs and maintenance associated with self-managed services.
Ignoring scalability and performance requirements that a single managed service can meet more efficiently.
Failing to consider operational risks in both options.

**Comparison:**

| Option          | Cost | Reliability | Operational Complexity |
|-----------------|------|-------------|------------------------|
| Managed Service | $80  | High        | Low                    |
| Self-Managed    | $100 | Medium      | High                   |

**Code:**

```python
# Example cost comparison code for budgeting
def compare_costs(managed_service_cost, self_managed_cost):
    if managed_service_cost <= self_managed_cost:
        return "Managed Service"
    else:
        return "Self-Managed Instances"

# Example usage
managed_service_cost = 80
self_managed_cost = 60 + 20 * 3  # Assuming each instance costs $20 and you need three

print(compare_costs(managed_service_cost, self_managed_cost))
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Budget: $100, Managed Service Cost: $80, Self-Managed Instance Cost Per Instance: $20, Number of Self-Managed Instances: 3 | Managed Service (Costs $80) |

**Data:**

```json
{
  "budget": 100,
  "managed_service_cost": 80,
  "self_managed_instances_cost_per_instance": 20,
  "number_of_self_managed_instances": 3
}
```

**Flow:**

1. Yes
2. No
3. Evaluate Cost
4. Within Budget
5. Exceeds Budget
6. Evaluate Reliability Needs
7. Managed Service?
8. Single Managed Service
9. Split Workloads AcrossInstances
10. Cost Comparison
11. Maintain Self-ManagedInstances for Customization

