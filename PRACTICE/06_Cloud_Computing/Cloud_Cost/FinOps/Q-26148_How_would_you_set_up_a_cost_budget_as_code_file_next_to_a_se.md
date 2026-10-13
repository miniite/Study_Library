---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a 'cost budget as code' file next to a service's config so exceeding it triggers a review? What lives in that file and who gets notified?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address setting up "cost budget as code" within a DevOps context, we need to establish a mechanism where cost thresholds are defined alongside service configurations, ensuring any breaches trigger automated reviews or alerts. At an intermediate level, this involves defining the budget constraints in a structured manner and integrating them with cloud cost governance tools.

#### Triage

The question is asking about setting up a "cost budget as code" file that will be used to monitor financial limits for a service within the DevOps workflow. The key elements include:

Defining the budget thresholds.
Ensuring these budgets are version-controlled and part of the codebase.
Automating notifications or reviews when costs exceed predefined thresholds.

#### First Principles Solution
Define Budgets: Create a YAML/JSON file that outlines cost limits for various services.
Integrate with Cost Governance Tools: Use tools like AWS Budgets, Azure Advisor, or Google Cloud Billing Alerts to enforce these budgets.
Automate Notifications and Reviews: Configure the tool to send alerts via email or Slack when costs exceed thresholds.
Version Control and CI/CD Integration: Ensure the budget file is part of your Git repository and integrated into your CI/CD pipeline for continuous monitoring.

#### Implementation
Steps

Create a Budget File

Define the budget parameters in a YAML/JSON file, e.g., service-budgets.yaml.
YAML
services:
  - name: webapp
    limit: 500 # in USD
  - name: database
    limit: 100 # in USD

Integrate with Cost Governance Tools

For AWS, use AWS Budgets to create budgets based on the values defined in service-budgets.yaml.
JSON
{
  "Budget": {
    "BudgetName": "webapp-cost-budget",
    "BudgetLimit": {
      "Amount": 500,
      "Unit": "USD"
    },
    "TimePeriod": {
      "Start": "2023-10-01T00:00:00Z",
      "End": "2023-10-31T00:00:00Z"
    }
  }
}

Automate Notifications

Configure AWS Budgets to send notifications via SNS when costs exceed the limit.
JSON
{
  "Version": "2008-10-17",
  "Id": "send-email-notification",
  "Statement": [
    {
      "Sid": "SendEmailOnBudgetExceed",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["sns:Publish"],
      "Resource": "<SNS_TOPIC_ARN>",
      "Condition": {
        "NumericLessThanEquals": {
          "budgets.amazonaws.com/CostAmount": 500
        }
      }
    }
  ]
}

Review Process

Set up a review process in your CI/CD pipeline to monitor the budget file and alert stakeholders.
YAML
steps:
  - name: MonitorBudgets
    action: ./scripts/check-budgets.sh
    env:
      BUDGET_FILE: service-budgets.yaml

Version Control and CI/CD

Ensure the service-budgets.yaml is included in your Git repository.
YAML
git:
  branches:
    only:
      - main
  hooks:
    pre-commit:
      - cmd: script/check-budget-file.sh
        name: "Check Budget File"

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By integrating cost budgets as code, we can ensure financial limits are consistently enforced across the organization. This approach leverages automation to send real-time notifications and triggers a review process when costs exceed thresholds.

#### Common Mistakes
Not version-controlling the budget file.
Failing to automate reviews or notifications.
Ignoring edge cases like cost fluctuations during promotions.

#### Follow-Ups
Budget Adjustments: How would you handle dynamic budget adjustments based on seasonal changes?
Cost Allocation: How do you ensure accurate cost allocation across teams and projects?
Compliance: What measures are in place to ensure regulatory compliance with financial reporting?

**Comparison:**

```
Service	Budget Limit (USD)
webapp	500
database	100
```

**Code:**

```
service-budgets.yaml

services:

name: webapp limit: 500 # in USD
name: database limit: 100 # in USD
CODE
Copy

```json
{
  "Budget": {
    "BudgetName": "webapp-cost-budget",
    "BudgetLimit": {
      "Amount": 500,
      "Unit": "USD"
    },
    "TimePeriod": {
      "Start": "2023-10-01T00:00:00Z",
      "End": "2023-10-31T00:00:00Z"
    }
  }
}
```

**Example input / output:**

```text
# Example check-budgets.sh output
Checking service-budgets.yaml...
Budget for webapp is within limit.
Budget for database is within limit.
No notifications triggered.
```

**Data:**

```json
{
  "SNS_TOPIC_ARN": "<your-SNS-topic-arn>"
}
```

**Flow:**

1. Service Launch
2. Code & Config Deployment
3. Budgets Applied as Code
4. Cost Monitoring via AWSBudgets
5. Notification on ExceedingLimit
6. Automated Review ProcessTriggered

