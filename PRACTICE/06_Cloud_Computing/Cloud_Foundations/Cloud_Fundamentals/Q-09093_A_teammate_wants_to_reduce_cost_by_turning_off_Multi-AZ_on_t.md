---
type: "practice"
difficulty: "Deep"
---

# A teammate wants to reduce cost by turning off Multi-AZ on the primary database. What single-AZ failure modes would you walk them through, and when is that trade actually acceptable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Availability Zone Resilience](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing with a teammate about turning off Multi-AZ (Multi-Availability Zone) on a primary database to reduce costs, it's crucial to understand the failure modes associated with single-AZ setups and when such a trade might be acceptable. Here’s a step-by-step breakdown:

TL;DR

Single-AZ databases lack redundancy across multiple availability zones, which can lead to more frequent downtime and data loss in case of zone failures or outages.

#### Triaging the Question

The question is probing whether you understand the failure modes associated with Single-AZ database setups compared to Multi-AZ configurations. It also seeks your judgment on when such a cost-saving measure might be acceptable despite potential risks.

#### Assumptions
The primary concern is operational availability and data integrity.
Cost reduction is a secondary but important factor.
There are no strict SLA (Service Level Agreements) or regulatory requirements that mandate Multi-AZ setups.

#### First Principles Solution

Single-AZ Failure Modes

Zone Outages: If an entire availability zone experiences an outage, the database will be unavailable until the zone is restored.
Maintenance and Updates: During planned maintenance or software updates in a single AZ, the database might go down temporarily.
Network Issues: Network failures within the same AZ can impact the database’s availability if it heavily relies on network resources.

Risk Assessment

Tolerable Downtime: Evaluate how much downtime is acceptable based on your application and business requirements. For instance, an e-commerce platform might have higher tolerances compared to a real-time trading system.
Impact Analysis: Assess the impact of downtime on user experience, data integrity, and financial implications. Consider both immediate and long-term effects.

Acceptability Criteria

Operational Impact: If the application can tolerate short periods of downtime (minutes to hours), then a Single-AZ setup might be acceptable.
Disaster Recovery: Evaluate your disaster recovery plans. Can you quickly failover or recover from outages in another data center, region, or cloud provider?
Cost-Benefit Analysis: Compare the cost savings with potential risks and recovery time objectives (RTOs) to ensure a balanced decision.

#### Closing Summary
+ Common Mistakes/Follow-Ups
TL;DR Recap: Single-AZ databases are less resilient compared to Multi-AZ setups, exposing your application to single-point failures. However, if the cost savings significantly outweigh the risk and downtime is tolerable, this trade-off might be acceptable.

#### Follow-Ups
RTO Planning: Ensure you have robust RTO plans in place for Single-AZ.
Disaster Recovery Testing: Regularly test disaster recovery processes to verify their effectiveness.
Monitoring and Alerting: Implement comprehensive monitoring and alerting to quickly detect and mitigate issues.

#### Common Mistakes
Overestimating the cost benefits without considering downtime impacts.
Underestimating the risk of a single zone outage leading to significant data loss or service disruptions.
Failing to properly plan for disaster recovery, which can negate any cost savings.

**Comparison:**

```
Failure Mode	Impact
Zone Outage	Database unavailable until zone is restored
Maintenance	Temporary database downtime during updates
Network Issues	Impact on availability if network resources are heavily relied upon
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Single-AZ Setup] --> B[Zone Outage];
    A --> C[Maintenance and Updates];
    A --> D[Network Issues];
```

**Example input / output:**

```text
Input: Current database setup is Multi-AZ, with a need to reduce costs.
Output: Single-AZ could be acceptable if the application can tolerate short periods of downtime and has robust RTO plans. Regular monitoring and testing are crucial.
```

**Data:**

```json
{
  "database_setup": {
    "current": "Multi-AZ",
    "proposed": "Single-AZ"
  },
  "failure_modes": [
    {"mode": "Zone Outage", "impact": "Database unavailable until zone is restored"},
    {"mode": "Maintenance and Updates", "impact": "Temporary database downtime during updates"},
    {"mode": "Network Issues", "impact": "Impact on availability if network resources are heavily relied upon"}
  ]
}
```

**Flow:**

1. Single-AZ Setup
2. Acceptable for short RTO
3. RTO Planning and Testing
4. Robust Recovery Plans
5. Maintain Monitoring andAlerting
6. Ensure Cost Savings JustifyRisks

