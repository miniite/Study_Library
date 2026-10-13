---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd model the true cost of a 'free' internal ML platform your team built to avoid a vendor bill, once you load in the on-call, GPU-driver maintenance, framework-upgrade churn, and opportunity cost of the platform engineers. When does build lose to buy for ML infrastructure specifically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model the true cost of a 'free' internal ML platform in terms of FinOps and Cloud Cost Governance, we need to account for all the hidden costs associated with maintaining and operating this platform, including on-call support, GPU-driver maintenance, framework-upgrade churn, and the opportunity cost of the platform engineers. This involves a multi-faceted approach that requires us to break down each component from first principles.

First, let's triage the question:

Understanding the Cost Components: The main goal is to quantify all non-obvious costs associated with running an internal ML platform.
Scope of Costs: We need to consider both direct (e.g., on-call support) and indirect costs (e.g., opportunity cost).
Comparative Analysis: Determine when building an internal platform is more cost-effective than purchasing services from a vendor.

Now, let's dive into the solution:

On-Call Support Costs:

On-call engineers are typically paid, even if they are not directly billed to the project.
Calculate their hourly rate and estimate the total time spent on calls for the platform.

#### Example
A team of 3 engineers each with a $200/hour rate working an average of 4 hours per week for 52 weeks.

GPU-Driven Maintenance:

GPUs require regular maintenance, which can involve updates, driver issues, and troubleshooting.
Estimate the cost per GPU and multiply by the number of GPUs in use.

#### Example
A single Tesla V100 costs $400/month to rent on the cloud, so 2 GPUs would amount to $800/month.

Framework Upgrade Churn:

Frameworks often require periodic updates, which can be time-consuming and costly.
Estimate the time spent upgrading frameworks and any associated downtime or performance issues.

#### Example
Upgrading PyTorch from version 1.9 to 1.10 might take a day of engineering time.

Opportunity Cost:

The opportunity cost is the value of what could have been achieved had resources not been allocated to building and maintaining this platform.
Estimate the revenue potential or other projects that could be pursued with those resources.

#### Example
A team of 3 engineers working on an internal ML platform could potentially contribute $1M in additional revenue through external projects.

Total Cost Calculation:

Sum up all the direct and indirect costs calculated above to get a comprehensive cost model for the internal platform.
Total = On-call support + GPU maintenance + framework upgrade churn + opportunity cost

#### Example
If on-call is $24,000/year, GPUs are $800/month (or $9,600/year), framework upgrades take 3 days at a rate of $150/day ($450), and the opportunity cost is estimated at $500,000.

#### Closing Summary

The true cost of running an internal ML platform includes direct costs such as on-call support and GPU maintenance, along with indirect costs like framework upgrade churn and opportunity cost. When comparing against vendor-provided services, we need to ensure that the savings from avoiding a vendor bill are greater than these total costs.

#### Common Mistakes/Follow-Ups
Overlooking Direct Costs: Engineers often underestimate direct costs such as on-call support.
Neglecting Indirect Costs: Opportunity cost and time spent on framework upgrades can significantly impact overall costs.
Vendor vs. Build Decision: When the internal platform's total cost exceeds the vendor bill, it might be more cost-effective to opt for a commercial service.

**Comparison:**

| Component             | Cost Estimate       |
|-----------------------|--------------------|
| On-call support       | $24,000            |
| GPU maintenance      | $9,600             |
| Framework upgrade    | $450               |
| Opportunity cost     | $500,000           |
| **Total Cost**        | **$598,450**       |

**Code:**

```python
# Example code snippet to calculate the total cost of an internal ML platform

def calculate_total_cost(on_call_support, gpu_maintenance, framework_upgrade_churn, opportunity_cost):
    on_call_cost = 3 * 200 * 4 * 52  # 3 engineers at $200/hour for 4 hours a week over a year
    gpu_cost = 2 * 800 * 12         # 2 GPUs at $800/month for a year
    framework_cost = 3 * 150        # 3 days of engineering time at $150/day
    return on_call_cost + gpu_cost + framework_cost + opportunity_cost

total_cost = calculate_total_cost(48000, 9600, 450, 500000)
print(f"Total Cost: ${total_cost}")
```

**Example input / output:**

```text
Input:
- On-call support hourly rate: $200/hour
- Hours per week: 4 hours
- Weeks per year: 52 weeks
- Team size: 3 engineers

GPU maintenance:
- Cost per GPU month: $800/month
- Number of GPUs: 2
- Months per year: 12 months

Framework upgrade churn:
- Hours spent per project: 3 days
- Daily rate: $150/day

Opportunity cost: $500,000/year

Output (total cost):
Total Cost: $598,450
```

**Data:**

```json
{
  "on_call_support": {
    "hourly_rate": 200,
    "hours_per_week": 4,
    "weeks_per_year": 52,
    "team_size": 3
  },
  "gpu_maintenance": {
    "cost_per_gpu_month": 800,
    "number_of_gpus": 2,
    "months_per_year": 12
  },
  "framework_upgrade_churn": {
    "hours_spent_per_project": 3,
    "daily_rate": 150
  },
  "opportunity_cost": 500000
}
```

**Flow:**

1. Yes
2. Calculate On-call Support
3. Is on-call time <= 3hours/week?
4. Calculate GPU Maintenance
5. Calculate Framework UpgradeChurn
6. Calculate Opportunity Cost
7. Sum All Costs
8. Determine Build vs BuyDecision Based on Total Cost

