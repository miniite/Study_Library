---
type: "practice"
difficulty: "Deep"
---

# How do you govern the cost of a self-service internal developer platform where teams provision databases, queues, and clusters with a click? What defaults, quotas, and cost-preview do you bake into the golden path?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the cost of a self-service internal developer platform, we need to establish a robust framework that balances flexibility with cost control. The key is to design default configurations, enforce quotas, and provide accurate cost previews throughout the provisioning process.

TL;DR

The solution involves setting up default resource templates with optimal cost-effective configurations, enforcing strict quotas at various levels (team, project), and integrating real-time cost forecasting tools that give developers visibility into potential costs before finalizing their requests. This ensures both developer productivity and financial control while enabling a self-service culture.

#### Triaging the Question

The question is asking how to manage the cloud costs associated with a self-service platform used by multiple development teams for provisioning various resources such as databases, queues, and clusters. The interviewer is probing on the implementation of governance policies, default settings, quota enforcement, and cost visibility mechanisms. Key assumptions include that all developers are familiar with basic cloud concepts but may not have deep knowledge about optimal resource usage.

#### First-Principles Solution

Default Configurations:

Define default templates for common resources like databases (e.g., RDS), queues (e.g., SQS), and clusters (e.g., Kubernetes).
Optimize these defaults to be cost-effective while still meeting typical development needs.

Quota Enforcement:

Implement per-team and project quotas to prevent excessive resource consumption.
Set up alerts for approaching or exceeding defined quotas, enabling proactive management.

Cost Preview:

Integrate real-time cost forecasting tools that provide developers with an estimated cost before finalizing their requests.
Use these previews to educate developers about the impact of their choices and encourage more informed decision-making.
Detailed Steps

Default Templates:

#### Example
For a typical developer database, use a t2.micro instance size instead of a c5.large, reducing costs by 60-70%.
Ensure default configurations balance between performance and cost-effectiveness.

Quota Enforcement:

At the team level, set limits on CPU, memory, and storage usage to prevent runaway spending.
For projects, introduce finer-grained controls based on project duration and scope.

#### Example
A team may have a monthly cap of $500 for RDS instances, with specific limitations on the number of instances or total storage.

Cost Preview:

Integrate cost forecasting tools like AWS Budgets, Azure Cost Management, or Google Cloud Pricing Calculator into the provisioning flow.
Provide developers with real-time estimates based on selected resources and configurations.

#### Example
If a developer chooses to provision an RDS instance, they receive a cost estimate of $50/month for 1GB storage.

#### Closing Summary

By implementing default templates, enforcing strict quotas, and providing detailed cost previews, we can ensure that developers have the flexibility to self-provision resources while maintaining financial control. This approach not only prevents costly mistakes but also educates developers about efficient resource usage practices.

#### Common Mistakes/Follow-Ups

Mistake: Focusing solely on minimizing costs without considering performance needs.

#### Follow-Up
Always balance cost with the necessary performance to avoid degraded developer productivity.

Mistake: Not providing adequate cost visibility, leading to surprise charges or inefficient resource usage.

#### Follow-Up
Ensure developers have access to accurate and timely cost information through built-in dashboards.

**Comparison:**

```
Resource Type	Default Configurations	Team Quotas (Monthly Limit)
RDS	t2.micro, 1GB storage	DevTeamA: $500
SQS	Standard queue with max 1,000 messages	DevTeamB: $300
Kubernetes	t2.small nodes, 4 CPU cores	All teams: $700
```

**Code:**

```python
# Example Python code for integrating AWS Budgets into a provisioning flow

import boto3

def setup_budget(account_id, budget_name, limit_amount):
    client = boto3.client('budgets')
    response = client.create_budget(
        AccountId=account_id,
        Budget={
            'BudgetName': budget_name,
            'Limit': {
                'Amount': limit_amount,
                'Unit': 'USD'
            },
            'TimePeriod': {
                'Start': '2023-10-01T00:00:00Z',
                'End': '2024-09-30T23:59:59Z'
            }
        }
    )
    return response
```

**Example input / output:**

```text
Input:
- Developer selects an RDS instance with t2.micro, 1GB storage

Output:
- Cost estimate is displayed: $50/month
- Developer can proceed or adjust the configuration based on the cost preview
```

**Data:**

```json
{
  "defaultRDSConfig": {
    "instanceType": "t2.micro",
    "allocatedStorageGB": 1,
    "storageType": "standard"
  },
  "teamQuotas": [
    {
      "teamName": "DevTeamA",
      "monthlyLimit": 500
    },
    {
      "teamName": "DevTeamB",
      "monthlyLimit": 300
    }
  ],
  "costForecastingData": [
    {
      "resourceType": "RDSInstance",
      "instanceConfig": {
        "instanceType": "t2.micro",
        "allocatedStorageGB": 1,
        "storageType": "standard"
      },
      "estimatedCost": 50
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Developer InitiatesProvisioning
6. Is default config used?
7. Resource is provisioned withdefaults
8. Select custom configuration
9. Cost preview calculated anddisplayed
10. Confirm resource allocation?
11. Resource is provisioned

