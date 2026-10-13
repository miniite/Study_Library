---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd quantify the 'reliability tax' of a specific managed cloud service versus self-hosting, over a three-year horizon, in a way a CFO and a staff engineer both trust.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quantify the "reliability tax" of a specific managed cloud service versus self-hosting, we need to consider multiple dimensions including uptime SLAs, disaster recovery capabilities, operational efficiency, and total cost of ownership (TCO). The key is to provide a holistic view that both a CFO and a staff engineer can understand and trust. Here’s a step-by-step approach:

Define the Metrics: Identify the key metrics for reliability such as Mean Time Between Failures (MTBF), Mean Time To Repair (MTTR), and Service Level Agreements (SLAs). For TCO, we need to consider both direct costs (like server hardware, software licenses) and indirect costs (human labor, support).

Estimate Costs: Calculate the initial setup cost for self-hosting versus the monthly or annual cost of a managed cloud service over a three-year horizon.

Calculate Uptime Costs: Compute the financial impact of downtime. For example, if a managed cloud service offers 99.95% uptime SLA and an SLO of 10 minutes MTTR, we can calculate the expected number of outages and their cost using historical data or service provider guarantees.

Operational Efficiency: Compare the operational overheads of self-hosting versus managed services. This includes the time spent on backups, updates, security patches, and monitoring. Managed services often reduce these costs significantly due to automation.

Disaster Recovery (DR) Capabilities: Evaluate DR capabilities in both scenarios. For self-hosting, this might involve setting up multiple data centers or using off-site storage solutions, which can be expensive. Managed cloud services typically offer built-in DR and failover mechanisms at a lower cost.

Service Level Agreements (SLAs): Assess the financial impact of non-compliance with SLAs. For example, if a service fails to meet its uptime guarantee, the provider often offers credits or refunds.

Human Capital: Consider the time and effort required from staff engineers in managing self-hosted environments versus leveraging managed services. This can include training costs, knowledge transfer, and real-time support availability.

Total Cost of Ownership (TCO): Sum up all direct and indirect costs for both scenarios over a three-year period to compare TCO. A detailed spreadsheet or tool like Excel or a specialized cloud cost management tool can be used here.

Risk Analysis: Quantify the risk associated with each option, including vendor lock-in, data sovereignty, and geopolitical risks. A risk matrix can help visualize these factors.

Scenario Testing: Conduct what-if analyses to test various scenarios such as unexpected outages or regulatory changes. This helps in understanding the worst-case and best-case TCOs for both options.

**Comparison:**

```
Scenario	Initial Setup Cost (Self-Hosted)	Annual Cloud Service Cost	Downtime Cost	Operation Overhead	Total TCO - Self-Hosted	Total TCO - Managed Cloud
Base	$50,000	$36,000	$10,000	$5,000	$91,000	$84,000
High Downtime	$50,000	$36,000	$20,000	$5,000	$107,000	$94,000

*Assumptions: 10% of MTTR as downtime cost and a single outage with a 1-hour impact.
```

**Code:**

```python
# Example Python code snippet to calculate TCO over a 3-year period

def calculate_tco(self_hosting_cost, cloud_service_cost, downtime_cost, operation_overhead):
    total_tco_self_hosting = self_hosting_cost + (downtime_cost * 0.1) # Assuming 10% of MTTR as downtime cost
    total_tco_cloud_service = cloud_service_cost
    return {"Self-hosting": total_tco_self_hosting, "Cloud Service": total_tco_cloud_service}

# Example input values (these are hypothetical)
self_hosting_cost = 50000 # Initial setup + annual maintenance
cloud_service_cost = 12000 * 3 # 4000 per year for 3 years
downtime_cost = 10000 # Cost of a single outage (1 hour at $10,000)
operation_overhead = 5000 # Annual operational overhead

# Calculate TCO
tco_result = calculate_tco(self_hosting_cost, cloud_service_cost, downtime_cost, operation_overhead)
print(tco_result)
```

**Example input / output:**

```text
| Scenario     | Initial Setup Cost (Self-Hosted) | Annual Cloud Service Cost | Downtime Cost | Operation Overhead | Total TCO - Self-Hosted | Total TCO - Managed Cloud |
|--------------|----------------------------------|--------------------------|---------------|--------------------|------------------------|--------------------------|
| Base         | $50,000                          | $36,000                  | $10,000       | $5,000             | $91,000                | $84,000                  |
| High Downtime | $50,000                          | $36,000                  | $20,000       | $5,000             | $107,000               | $94,000                  |

*Assumptions: 10% of MTTR as downtime cost and a single outage with a 1-hour impact.
```

**Data:**

```json
{
    "self_hosting_cost": 50000,
    "cloud_service_cost": 12000 * 3,
    "downtime_cost": 10000,
    "operation_overhead": 5000
}
```

**Flow:**

1. Yes
2. No
3. Define Metrics
4. Is the Metric Critical?
5. Calculate Financial Impact
6. Skip to Next Step
7. Estimate Cost Over 3 Years
8. Perform Scenario Testing
9. Risk Analysis
10. Collapse to TCO
11. Operational Efficiency & DRCapabilities
12. Calculate TCO for Both Options

