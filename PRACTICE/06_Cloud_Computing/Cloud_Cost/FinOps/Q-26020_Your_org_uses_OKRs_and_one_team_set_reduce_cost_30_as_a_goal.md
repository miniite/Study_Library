---
type: "practice"
difficulty: "Expert"
---

# Your org uses OKRs and one team set 'reduce cost 30%' as a goal, then achieved it by simply shrinking their product's scope. How do you design cost OKRs that reward efficiency, not just doing less? What's the guardrail metric?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Healthy vs Wasteful Cost Growth](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Healthy_vs_Wasteful_Cost_Growth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge in designing cost OKRs (Objectives and Key Results) that incentivize efficiency rather than merely cutting scope, we need to focus on metrics that measure productivity and value preservation while acknowledging the trade-offs of reducing costs. The key is to ensure that teams are rewarded for optimizing resource use without compromising product quality or customer experience.

#### Triage

The core issue here involves setting up a system where cost reduction is achieved through efficient use of resources, rather than just cutting scope. This requires designing OKRs and guardrail metrics that align with broader business objectives while providing clear incentives for efficiency and optimization.

Step-by-Step Solution

Define Objectives:

Objective: Reduce operational costs by 30% over the next quarter.
Ensure this objective is aligned with long-term strategic goals, such as maintaining or improving product quality and customer satisfaction.

Key Results (KRs):

Implement automated cost monitoring tools to track cloud resource usage in real-time.
Conduct regular code reviews to identify inefficient code that could be optimized.
Introduce a cost-saving challenge for the team with rewards based on performance metrics.

Guardrail Metrics:

Cost per Feature: Measure the operational cost of each new feature added or improved. This ensures that while costs are reduced, there is no penalty for adding value to the product.
Customer Satisfaction Score (CSAT): Track customer feedback and satisfaction levels before and after changes in resource usage.
Resource Utilization Efficiency: Use metrics like CPU utilization, memory efficiency, and disk I/O to ensure that resources are being used effectively.
Detailed Example

Let's say the team has a baseline of $500 monthly operational cost for maintaining their product features. Here’s how you could set up some KRs:

Objective: Reduce operational costs by 30%.
Key Results:
Implement and maintain an automated cloud monitoring tool to track cost usage, achieve a reduction of at least $150 in the next quarter (40%).
Conduct code reviews every two weeks; each code review must identify at least one inefficiency that can be improved.
Launch a cost-saving challenge where teams earn points for identifying and implementing cost-saving measures.

#### Closing Summary

Designing cost OKRs that reward efficiency involves creating balanced objectives, key results, and guardrail metrics. By focusing on productivity and value preservation, we ensure that teams are incentivized to find the right balance between reducing costs and maintaining product quality. Guardrails like CSAT scores help monitor impact on customer experience.

#### Common Mistakes/Follow-Ups

Mistake: Focusing solely on cost reduction without considering the quality of deliverables.

#### Follow-Up
Ensure that KRs include metrics for feature value and customer satisfaction to balance efficiency with product health.

Mistake: Over-relying on scope reduction as a solution.

#### Follow-Up
Use metrics like resource utilization and CSAT scores to ensure teams are optimizing usage, not just cutting scope.

**Comparison:**

| Guardrail Metrics | Description |
|-------------------|-------------|
| Cost per Feature   | Measure the operational cost of each new feature added or improved. |
| Customer Satisfaction Score (CSAT) | Track customer feedback and satisfaction levels before and after changes in resource usage. |
| Resource Utilization Efficiency | Use metrics like CPU utilization, memory efficiency, and disk I/O to ensure that resources are being used effectively. |

**Code:**

```python
# Example of a simple cost tracking script using AWS CloudWatch API
import boto3

def get_cost_data():
    client = boto3.client('ce')
    response = client.get_cost_and_usage(
        TimePeriod={
            'Start': '2023-01-01',
            'End': '2023-04-01'
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost']
    )
    return response['ResultsByTime'][0]['Total']['UnblendedCost']['Amount']

cost = get_cost_data()
print(f"Monthly Operational Cost: ${cost}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Baseline Cost Data | $500 monthly operational cost |
| KRs Achievement | Achieved cost reduction of $150, identified 3 inefficiencies, and won the cost-saving challenge with 10 points |

**Data:**

```json
{
  "objectives": [
    {
      "objective": "Reduce operational costs by 30% over the next quarter",
      "metrics": [
        "Cost per Feature",
        "Customer Satisfaction Score (CSAT)",
        "Resource Utilization Efficiency"
      ]
    }
  ],
  "key_results": [
    "Implement automated cost monitoring tool to track cost usage, achieve a reduction of at least $150 in the next quarter (40%)",
    "Conduct code reviews every two weeks; each code review must identify at least one inefficiency that can be improved",
    "Launch a cost-saving challenge where teams earn points for identifying and implementing cost-saving measures"
  ]
}
```

**Flow:**

1. Define Objectives
2. Set Key Results (KRs)
3. Implement Guardrail Metrics
4. Monitor Performance
5. Adjust KRs if Necessary
6. Achieve Objective

