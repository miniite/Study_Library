---
type: "practice"
difficulty: "Deep"
---

# You're evaluating whether to migrate a stable, predictable workload from a hyperscaler to a cheaper neo-cloud (Hetzner, OVH, bare metal). Build the honest risk-adjusted TCO including the exit-cost and reliability delta.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to migrate a stable, predictable workload from a hyperscaler like AWS or Google Cloud to a cheaper neo-cloud provider such as Hetzner or OVH, we need to consider several factors beyond just the initial cost of hosting. The key is to build an honest risk-adjusted Total Cost of Ownership (TCO) analysis that includes exit costs and reliability differences. This process involves understanding the current infrastructure, forecasting future needs, and evaluating the total cost over a multi-year period.

Firstly, let's break down what TCO encompasses:

Initial Hosting Costs: The cost of setting up and running the environment.
Operational Costs: Ongoing maintenance, monitoring, and support.
Exit Costs: Penalties or costs associated with leaving one provider for another.
Reliability Differences: Potential downtime risks and recovery times.

To build this analysis, we'll follow these steps:

Assess the current infrastructure.
Forecast future requirements.
Evaluate the cost of moving to the new environment.
Calculate the risk-adjusted TCO over a typical 5-year period.
Compare the costs between staying with the current provider and migrating.

Let's assume we are currently on AWS EC2 instances, and our application is stable with predictable usage patterns. We will evaluate whether switching to Hetzner Cloud, which offers competitive pricing, might be more cost-effective in the long run.

Step-by-Step Analysis

Assess Current Infrastructure:

Review current instance types, network configurations, storage needs.
Identify any custom configurations or third-party tools.

Forecast Future Requirements:

Estimate future growth in terms of CPU, memory, and storage.

#### Cons
ider expected changes in usage patterns (e.g., seasonal fluctuations).

Evaluate the Cost of Moving to Hetzner Cloud:

Research pricing models for Hetzner Cloud, including instance types, network costs, and any additional services needed.
Calculate potential savings or increased costs.

Calculate Risk-Adjusted TCO:

Identify exit penalties from AWS (if applicable).
Assess reliability differences: uptime guarantees, disaster recovery capabilities.

#### Cons
ider other factors like regulatory compliance and data transfer costs.

Compare Costs Over 5 Years:

Develop a detailed cost model that includes initial setup, ongoing operations, potential downtime impacts, and exit costs.

#### Example
TCO Calculation

Assume the current AWS TCO per year is $10,000 based on instance types, storage, networking, and maintenance. The projected growth rate suggests an increase of 2% annually in usage.

Moving to Hetzner Cloud:
Initial setup cost: $500 (assuming minimal migration complexity).
Annual hosting costs: $8,000 (based on competitive pricing for similar instance types).

Exit penalties from AWS: $1,000. Reliability differences: Additional downtime could impact business operations.

Risk-Adjusted TCO

To calculate the risk-adjusted TCO, we need to consider the probability and impact of potential disruptions. For example, if there is a 2% chance of significant downtime due to reliability issues, the impact on the business would be $50,000 in lost revenue.

Exit-Cost Adjustment:

[ \text{Exit Cost} = 1% \times (Initial Hosting Costs + Operational Costs) + Exit Penalties ]

Reliability Delta:

[ \text{Reliability Impact} = P(\text{Down Time}) \times (\text{Impact on Business}) ]

Final TCO Comparison
AWS: ( $10,000 \text{ initial cost} + $8,000 \text{ ongoing costs} + $500 \text{ exit penalties} + 2% \times (Cost of Downtime) )
Hetzner Cloud: ( $500 \text{ setup cost} + $8,000 \text{ ongoing costs} - $1,000 \text{ exit penalties} + 2% \times (Cost of Downtime) )

#### Common Mistakes
and Follow-Ups
Ignoring Exit Costs: Many companies fail to factor in the cost associated with exiting a cloud provider.
Overlooking Reliability Differences: The reliability of a cloud provider can significantly impact business operations.
Not Factoring in Future Growth: Current costs might not fully capture future scalability needs.

In conclusion, the honest risk-adjusted TCO analysis should consider all financial and operational impacts to make an informed decision. A thorough evaluation will help identify whether the potential cost savings outweigh the risks associated with migration.

**Code:**

```python
# Example Python code for a simple TCO calculation
def calculate_tco(current_hosting_cost, projected_growth_rate, exit_penalty):
    initial_cost = 500
    ongoing_cost = current_hosting_cost * (1 + projected_growth_rate)

    # Exit cost with penalties
    exit_cost = initial_cost + ongoing_cost + exit_penalty

    return exit_cost

# Example usage
current_hosting_cost = 8000  # Annual hosting costs on Hetzner Cloud
projected_growth_rate = 0.02  # 2% annual growth in usage
exit_penalty = 1000  # Exit penalty from AWS

total_tco = calculate_tco(current_hosting_cost, projected_growth_rate, exit_penalty)
print(f"Total TCO over 5 years: {total_tco}")
```

**Example input / output:**

| Current Provider | New Provider |
|------------------|--------------|
| AWS              | Hetzner      |
| Initial Cost     | $10,000      | $500 (setup) + $8,000 ongoing - $1,000 exit penalties |
| Projected Growth Rate | 2% annual growth in usage | 2% annual growth in usage |
| Exit Penalty     | $1,000       | $1,000          |

**Data:**

```yaml
# Example data structure for cost analysis
cost_analysis:
  current_provider: aws
  new_provider: hetzner
  initial_cost: 10000  # Current AWS TCO per year
  projected_growth_rate: 0.02  # 2% annual growth in usage
  exit_penalty: 1000  # Exit penalty from AWS
  ongoing_cost_hetzner: 8000  # Annual hosting costs on Hetzner Cloud
```

