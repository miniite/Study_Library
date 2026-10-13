---
type: "practice"
difficulty: "Deep"
---

# Explain the risk of committing to a 3-year Savings Plan on an instance family you might migrate off of. How do you hedge that regret when the compute shape is uncertain?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Commitments and Spot Capacity](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When considering a 3-year Savings Plan commitment in AWS, especially on an instance family that you might potentially migrate away from, there are significant financial and operational risks involved. This commitment ties up substantial capital, which could be lost if the chosen instance type is not optimal or if migration plans change. To mitigate this risk, it's crucial to implement robust cost governance strategies and use tools like AWS Cost Explorer and budget alerts.

#### Triaging the Question

The core of this question revolves around understanding the financial commitments involved in committing to a 3-year Savings Plan for an instance family that may not be your long-term solution. The interviewer is probing your knowledge of cloud cost optimization, risk management, and strategic decision-making capabilities. Assumptions include:

You are familiar with AWS pricing models and services.
You understand the benefits and risks associated with Savings Plans.

#### First Principles Solution

To start, let's break down the problem from first principles:

Understanding Savings Plans:

A 3-year Savings Plan in AWS is a commitment-based offering that provides discounts on EC2 Reserved Instances (RIs).
The upfront cost of the plan is significant; for instance, a $500 monthly savings plan might require an upfront payment of around $14,000.

Risk of Commitment:

If the chosen instance family becomes obsolete or if you need to migrate away from it, the upfront costs are non-refundable.
There could be unexpected changes in your workload that make the committed instances unsuitable, leading to suboptimal resource utilization and unnecessary expenses.

Mitigation Strategies:

Implementing cost governance and monitoring tools like AWS Cost Explorer can help you track spending and identify potential issues early.
Using budget alerts in AWS can notify you when costs exceed a certain threshold, allowing for timely corrective actions.
Regularly reviewing and re-evaluating your instance families to ensure they align with current needs.

Hedging Regret:

Develop an exit strategy that allows for flexibility in case the migration plan changes. This could involve:
Setting up a reserve capacity or on-demand instances as a fallback.
Using AWS Budgets and Cost Allocation Tags to monitor and control spending.
Leveraging AWS Savings Plans Reserve Conversion Tool, which helps you transition out of committed savings plans without incurring upfront fees.

#### Closing Summary

Committing to a 3-year Savings Plan for an instance family that may be migrated off is inherently risky due to the high upfront cost and the potential mismatch between current and future needs. To mitigate this risk, leveraging robust cost governance tools, implementing budget alerts, regularly reviewing your infrastructure decisions, and having a flexible exit strategy are essential steps.

#### Common Mistakes/Follow-Ups
Mistake: Overcommitting to Savings Plans without proper analysis of long-term needs.

#### Follow-Up
Discussing how AWS Reserve Conversion Tool can be used effectively. Explore other cost optimization strategies like Spot Instances, Dedicated Hosts, and AWS Marketplace.

**Comparison:**

```
Risk Factor	Mitigation Strategy
High Upfront Cost	Reserve Conversion Tool
Uncertain Workload	Regular Reviews
Potential Mismatch	Exit Strategies (On-Demand, Spot)
```

**Code:**

```python
# Example Python code snippet for setting up AWS Budgets using Boto3
import boto3

def create_aws_budget(budget_name, budget_amount):
    client = boto3.client('budgets')
    response = client.create_budget(
        AccountId='123456789012',  # Replace with your AWS account ID
        Budget={
            'BudgetName': budget_name,
            'TimeUnit': 'MONTH',
            'BudgetType': 'COST',
            'BudgetLimit': {
                'Amount': budget_amount,
                'Unit': 'USD'
            },
            'TimePeriod': {
                'Start': '2023-01-01T00:00:00Z',  # Replace with desired start date
                'End': '2024-01-01T00:00:00Z'   # Replace with desired end date
            }
        },
        NotificationsWithServiceRole={
            'NotificationTypes': ['ACTUAL'],
            'NotificationDestination': 'arn:aws:sns:us-west-2:123456789012:my-budget-sns-topic'
        }
    )
    print(response)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Committing to a 3-year Savings Plan for General1 instance family with $50k upfront cost, but considering potential migration away from it. | Need robust cost governance (AWS Cost Explorer), budget alerts, regular reviews, and exit strategies like on-demand or Spot Instances. |

**Data:**

```json
{
  "SavingsPlanDetails": {
    "SavingsPlanName": "MyElasticComputeSavingsPlan",
    "SavingsPlanTermInYears": 3,
    "InstanceFamily": "General1",
    "TotalCommitmentAmount": 50000.0,
    "DiscountedRatePerUnit": 0.07
  }
}
```

**Flow:**

1. Initiate Savings Plan
2. Cost Governance Setup
3. Budget Alerts
4. Regular Reviews
5. Exit Strategies (On-Demand,Spot)

