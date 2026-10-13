---
type: "practice"
difficulty: "Deep"
---

# A team keeps hitting its budget cap and quietly moves work to a personal cloud account. How would you detect shadow spend and what governance keeps it inside the org?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shadow IT and Budget Evasion](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_IT_and_Budget_Evasion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of "shadow spend" within a DevOps team, we need to implement comprehensive FinOps (Financial Operations) practices combined with robust cloud cost governance mechanisms. This involves detecting unauthorized or unapproved spending on personal cloud accounts and ensuring that all cloud usage stays under the organization's budget constraints.

Firstly, it’s important to understand what "shadow spend" means: it refers to cloud expenses incurred by employees using their own accounts without proper authorization or tracking. This can lead to financial inefficiencies and security risks. To combat this, we need a multi-layered approach that includes technical measures, policy enforcement, and monitoring tools.

#### Triage

The core question is how to detect and manage "shadow spend" effectively. The interviewer is probing our understanding of cloud cost governance, the importance of financial visibility, and the implementation of internal controls. We also need to consider the trade-offs between security, compliance, and operational efficiency.

#### Assumptions
The organization has a budget for cloud resources.
There are multiple teams using cloud services, including a DevOps team.
Personal cloud accounts may be used by individuals within the organization.
Financial and security policies are in place but may not be strictly enforced.

#### First-Principles Solution

Implement Cost Management Tools:

Utilize AWS Budgets, Azure Costs, or GCP Cost Management tools to set up budget alerts and cost reports.
Set up thresholds that trigger notifications when a team approaches its spending limit.

Enforce Identity and Access Management (IAM):

Ensure all cloud accounts are managed via centralized IAM services like AWS IAM, Azure AD, or Google Cloud IAM.
Implement strict access controls to prevent unauthorized account creation and usage.

Use Cost Allocation Tags:

Tag all resources with relevant metadata such as department, project, and cost center.
This helps in accurately attributing costs to specific teams and projects.

Automate Monitoring and Alerting:

Set up automated monitoring tools that can detect unusual spending patterns.
Implement alert mechanisms for when budget thresholds are exceeded or when new accounts are created without approval.

Regular Audits and Compliance Checks:

Conduct periodic audits to ensure compliance with financial policies.
Perform internal reviews of cloud usage reports to identify discrepancies.

Educate and Train Teams:

Provide training on proper cloud usage practices and the importance of cost management.
Ensure all team members understand the financial implications of their actions.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing a combination of technical, policy-driven, and educational measures, we can effectively manage "shadow spend" and ensure that cloud costs remain under control. Common mistakes include relying solely on technology without addressing cultural or procedural issues. It’s crucial to balance security with ease of use and to continuously monitor and refine the governance framework.

#### Follow-ups
might involve discussing specific tools, integration strategies, and how these solutions can be scaled across different teams within a larger organization.

**Comparison:**

| Cost Category | Description            |
|---------------|------------------------|
| EC2           | Amazon Elastic Compute Cloud |
| S3            | Simple Storage Service    |
| RDS           | Relational Database Service|

**Code:**

```python
# Example Python script for sending budget alerts using AWS Budgets API
import boto3

def create_budget(client, account_id, budget_name, threshold):
    response = client.create_budget(
        AccountId=account_id,
        Budget={
            'BudgetName': budget_name,
            'TimeUnit': 'DAYS',
            'BudgetLimit': {
                'Amount': threshold,
                'Unit': 'USD'
            },
            'CostFilters': [
                {
                    'Dimensions': {
                        'Key': 'SERVICE',
                        'Values': ['EC2']
                    }
                }
            ],
            'NotificationSettings': {
                'NotifyWhen': 'ACTUAL',
                'Subscribers': [
                    {
                        'SubscriptionType': 'EMAIL',
                        'Address': 'devops@example.com'
                    }
                ]
            }
        }
    )
    return response
```

**Example input / output:**

```text
Input:
- Create a budget for the DevOps team with an alert threshold of $5000 USD per day.

Output:
{
  "BudgetArn": "arn:aws:ce:us-east-1:123456789012:budget/DevOpsTeamCosts",
  "AccountId": "123456789012"
}
```

**Data:**

```json
{
  "Budget": {
    "BudgetName": "DevOpsTeamCosts",
    "TimeUnit": "DAYS",
    "BudgetLimit": {
      "Amount": 5000,
      "Unit": "USD"
    },
    "CostFilters": [
      {
        "Dimensions": {
          "Key": "SERVICE",
          "Values": ["EC2"]
        }
      }
    ],
    "NotificationSettings": {
      "NotifyWhen": "ACTUAL",
      "Subscribers": [
        {
          "SubscriptionType": "EMAIL",
          "Address": "devops@example.com"
        }
      ]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Detect Unusual Spending
4. Is Budget Exceeded?
5. Alert Team and Management
6. Monitor Regularly for Trends
7. Review and Take Action
8. Continue Monitoring

