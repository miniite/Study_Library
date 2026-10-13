---
type: "practice"
difficulty: "Intermediate"
---

# How would you explain to a product manager why moving your compute to a cheaper region across the country could raise your total cost once you count cross-region data transfer to the database?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Egress and Data Transfer Costs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Egress_and_Data_Transfer_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing cloud fundamentals with a product manager, it's crucial to explain that while moving compute resources to a more economical region might seem like an immediate cost-saving measure, additional costs related to cross-region data transfer can significantly offset these savings. The key is understanding the network and storage costs associated with this migration.

In essence, we need to evaluate not only the initial cost of changing regions but also consider the ongoing expenses for moving data between regions. Here’s a step-by-step breakdown:

Initial Cost Analysis: Assess the cost of moving your compute resources to a cheaper region.
Data Transfer Costs: Calculate the costs associated with transferring existing and new data from the current region to the target region.
Ongoing Network Costs: Estimate the ongoing network costs for any data that needs to be transferred between regions, including read and write operations.

Let’s walk through each of these steps in detail:

TL;DR

Moving compute resources to a cheaper region can initially appear cost-effective, but cross-region data transfer incurs significant additional costs. These include one-time migration costs and ongoing network charges for data movement.

#### Triaging the Question

The core of this question is about understanding the total cost implications of moving your infrastructure across regions in cloud services like AWS or Azure. The key points to cover are:

Initial Cost Analysis: Evaluate if the compute resources in a cheaper region are indeed more economical.
Data Transfer Costs: Understand that data stored in one region and accessed from another will incur charges, which can be substantial depending on the amount of data and frequency of access.
Ongoing Network Costs: Consider the continuous network costs for transferring data between regions.

#### Assumptions
The compute resources are currently hosted in a more expensive region.
There is existing data that needs to be moved to the new region.
Regular data transfers will occur due to application requirements or user interactions.

#### First-Principles Solution
Step 1: Initial Cost Analysis

First, we need to compare the cost of compute resources in both regions:

Current Region Costs: Assume the current region costs $0.50 per hour for a particular instance type.
Target Region Costs: The target region might offer cheaper rates at $0.35 per hour.
Step 2: Data Transfer Costs

Next, we need to estimate the cost of transferring data between regions:

One-Time Migration Costs:

Assume you have 1 TB of existing data.
AWS S3 Cross-Region Replication (XRR) or similar service might charge a one-time transfer fee. For example, AWS charges $0.50 per GB for the first 50 TB and then $0.20 per GB thereafter.

Ongoing Network Costs:

For regular data transfers, you’ll incur ongoing network costs.
Assume an average of 10 GB of data transferred daily between regions.
AWS charges around $0.09 per GB for data transfer out of the region and $0.085 per GB for data transfer into the region.
Step 3: Ongoing Network Costs

Finally, we need to consider the ongoing network costs:

Daily Data Transfer: Calculate daily costs based on the transfer rates.
Monthly Data Transfer: Multiply by 30 days to get a monthly cost.

Let’s do some calculations:

One-time migration cost for 1 TB: $50 (first 50 TB) + $20 = $70
Daily network cost out: 10 GB * $0.09 = $0.90
Daily network cost in: 10 GB * $0.085 = $0.85
Total daily network cost: $0.90 + $0.85 = $1.75

Monthly data transfer cost: $1.75/day * 30 days = $52.50

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while moving to a cheaper region can reduce initial compute costs, the one-time migration and ongoing network costs must be considered. A detailed analysis is needed to ensure that the total cost does not outweigh the savings.

#### Common Mistakes

Overlooking data transfer fees.
Failing to consider long-term network usage patterns.
Not accounting for potential performance impacts of cross-region operations.
Common Follow-Ups
Can we optimize data access patterns? (e.g., caching, local storage)
Are there ways to reduce migration costs? (e.g., use incremental transfers during off-peak hours)

**Comparison:**

```
Cost Component	Amount
Initial migration fee	$70
Daily network out cost	$0.90
Daily network in cost	$0.85
Total daily network cost	$1.75
Monthly network cost	$52.50
```

**Code:**

```python
# Example Python code for estimating one-time and ongoing network costs
data_size_gb = 1024  # 1 TB data
one_time_transfer_fee = 50 + 20  # $70 for AWS XRR

daily_out_cost = (data_size_gb / 8) * 0.09  # 10 GB/day, $0.09/GB out
daily_in_cost = (data_size_gb / 8) * 0.085  # 10 GB/day, $0.085/GB in

total_daily_network_cost = daily_out_cost + daily_in_cost
monthly_network_cost = total_daily_network_cost * 30

print(f"One-time transfer cost: ${one_time_transfer_fee}")
print(f"Monthly network cost: ${monthly_network_cost:.2f}")
```

**Example input / output:**

```text
Input:
- Existing data size: 1 TB (1,024 GB)
- Daily transfer volume: 10 GB

Output:
One-time transfer cost: $70
Monthly network cost for transfers: $52.50
```

**Data:**

```json
{
  "current_region": {
    "cost_per_hour": "$0.50",
    "data_transfer_out_rate": "$0.09/GB"
  },
  "target_region": {
    "cost_per_hour": "$0.35",
    "data_transfer_in_rate": "$0.085/GB"
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Region Analysis
4. Compare Cost Per Hour
5. Are we moving existing data?
6. Migrate Existing Data
7. Migrate and CalculateOne-Time Costs
8. Calculate Ongoing NetworkCosts
9. Collapse into Total MonthlyCosts
10. Summarize Total Costs

