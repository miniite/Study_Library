---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks about incentive design: your best engineers optimize what's measured, so the moment you publish a cost leaderboard, teams start gaming it by moving cost off their tags. How do you design a cost-visibility system that's hard to game by construction rather than by policing?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Funding Models and Incentives](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Funding_Models_and_Incentives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR

To create an effective FinOps and cloud cost governance system that discourages gaming, we need to design a cost visibility framework that aligns team incentives with broader business objectives. This involves leveraging automated cost allocation, transparent reporting, and multi-dimensional cost breakdowns to ensure fairness and reduce the incentive for teams to manipulate costs.

#### Triaging the Question

The core of this question is about designing a system that prevents gaming behavior by teams who might try to misreport their cloud costs to look better than they are. The goal is to create an environment where transparency and accountability lead to true cost optimization, rather than artificial metrics manipulation. We need to consider several factors: automated cost allocation accuracy, team incentives, and the overall organizational culture.

Key Assumptions
Cost Allocation Accuracy: Automated tools must provide precise and accurate cost allocations.
Team Incentives: Teams are incentivized based on their cloud spending.
Organizational Culture: There is a willingness to embrace transparency and responsibility for costs.
Scalability: The system needs to scale with the number of teams and resources.
Step-by-Step Solution
1. Automated Cost Allocation

The first step in designing an effective cost governance system is ensuring that cloud provider APIs are leveraged to accurately allocate costs based on tagging, usage patterns, and other relevant metrics. Tools like AWS Budgets or Azure Cost Management provide built-in features for automatic cost allocation.

Tag-based Allocation
AWS Budgets
Azure Cost Management
2. Transparent Reporting

Transparency is key to preventing gaming behavior. Teams should see detailed and consistent reports that break down their costs in real-time. These reports should include not only the total cost but also granular details like resource usage, billable hours, and SaaS subscriptions.

Real-time Reports
Cost Breakdown
3. Multi-Dimensional Cost Breakdowns

To discourage gaming, we need to create a multi-dimensional cost breakdown that includes various metrics such as project-specific costs, resource utilization, and time-based expenses. This approach ensures that teams cannot easily manipulate their overall cost by shifting usage or tagging.

Metric-based Allocation
Project-Specific Costs
Resource Utilization
Time-based Expenses
4. Incentive Alignment with Business Goals

Incentives should be designed to align directly with business goals rather than just cost savings. For example, teams could be rewarded for reducing idle time on resources or optimizing cloud-native services like Lambda functions.

Business Goals
Cost Savings
Resource Optimization
5. Regular Audits and Feedback Loops

Regular audits can help catch any attempts to game the system, but these should be part of a broader feedback loop where teams are encouraged to understand their costs better and make informed decisions.

Audits
Feedback Loops

#### Closing Summary

Designing a cost-visibility system that is hard to game by construction involves a combination of automated, accurate cost allocation, transparent reporting, multi-dimensional breakdowns, aligned incentives, and regular feedback loops. By focusing on these elements, we can create an environment where teams are motivated to optimize their cloud usage without the need for constant policing.

#### Common Mistakes
and Follow-Ups

Mistakes:

Over-reliance on Manual Processes: This can lead to inaccuracies and gaming behavior.
Lack of Granularity in Cost Breakdowns: Failing to provide detailed insights into costs makes it harder to identify potential issues.
Ignoring Team Incentives: Without aligning incentives with broader business goals, teams may not be motivated to participate.

#### Follow-Ups

Discuss specific tools and technologies that can be used for cost allocation and reporting.
Explore examples of organizations that have successfully implemented such systems.

#### Cons
ider how the system scales as more resources are added or removed.

**Comparison:**

```
2023-10-01
2023-10-08
2023-10-15
2023-10-22
2023-10-29
2023-11-05
2023-11-12
Marketing-Prod
Development-QA
Project Costs
Example Gantt Chart for Cost Allocation
```

**Code:**

```python
# Example Python code to fetch AWS Cost data using Boto3

import boto3

def get_aws_cost_data(tag_key, tag_value):
    client = boto3.client('budgets')

    response = client.describeBudgetActionsForCostAlarms(
        CostType='UNBLURRED',
        TimePeriod={
            'Start': '2023-10-01T00:00:00Z',
            'End': '2023-10-31T23:59:59Z'
        },
        Filters=[
            {
                'Type': 'DIMENSION',
                'Key': tag_key,
                'Values': [tag_value],
                'ComparisonOperator': 'EQUALS'
            }
        ]
    )

    return response['BudgetActions']
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Fetch cost data for Marketing project tagged with 'Environment': Prod | Cost Data for Marketing - Prod Environment: \n- EC2: $50\n- S3: $450 \nTotal Cost: $500 |

**Data:**

```json
{
  "costData": [
    {
      "tagKey": "Project",
      "tagValue": "Marketing",
      "cost": 500,
      "resources": ["EC2", "S3"]
    },
    {
      "tagKey": "Environment",
      "tagValue": "Prod",
      "cost": 1500,
      "resources": ["Lambda", "RDS"]
    }
  ]
}
```

**Flow:**

1. Tag-based Allocation
2. AWS Budgets
3. Transparency in Reporting
4. Multi-dimensional Breakdown
5. Incentive Alignment
6. Audits and Feedback Loops

