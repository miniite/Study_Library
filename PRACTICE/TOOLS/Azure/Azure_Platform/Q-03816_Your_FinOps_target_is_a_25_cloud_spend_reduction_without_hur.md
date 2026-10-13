---
type: "practice"
difficulty: "Expert"
---

# Your FinOps target is a 25% cloud spend reduction without hurting reliability; describe the org-level operating model (showback/chargeback, commitment strategy, guardrails) you'd stand up over two quarters.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Azure Messaging and Reliability](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/Azure_Messaging_and_Reliability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve a 25% cloud spend reduction while maintaining reliability in an organization, we need to establish a robust operating model with a combination of showback/chargeback mechanisms, commitment strategies, and guardrails. Over two quarters, the process will involve setting up these components, gradually rolling them out, and continuously optimizing based on feedback.

TL;DR

The key steps are:

Define Showback/Chargeback Mechanisms: Implement cost allocation to individual teams or departments.
Establish Commitment Strategy: Set realistic targets for each team to meet the overall 25% reduction goal.
Implement Guardrails: Enforce spending limits and prevent overspending through budget controls.

#### Triage

The question is asking how to structure an organization's FinOps practices to achieve a significant cost reduction while ensuring that critical services remain reliable. The core focus is on setting up a fair but effective financial accountability framework and enforceable policies.

#### First-Principles Solution
1. Define Showback/Chargeback Mechanisms

The first step is to implement a clear showback or chargeback system that allocates cloud costs accurately across teams. This can be done using Azure Cost Management and Billing tools, which provide detailed cost reports broken down by resource groups, subscriptions, and user-defined tags.

Showback Model: Allocate costs based on usage metrics (e.g., VM hours, storage bytes) to each team.
Chargeback Model: Implement a more granular model where teams are charged for their cloud consumption directly. This can be enforced via Azure's budgeting and alerting features.
2. Establish Commitment Strategy

Once the cost allocation framework is in place, define clear performance targets for each team. These targets should align with both cost reduction goals and reliability requirements.

Quarter 1: Set initial baseline costs and determine expected savings.
Quarter 2: Adjust budgets and introduce stricter controls to ensure teams meet their commitments.
3. Implement Guardrails

Guardrails are critical to prevent overspending and ensure resource utilization is optimized.

Budgeting: Use Azure Budgets to set spending limits for each team and subscription.
Alerting: Configure alerts to notify stakeholders when budget thresholds are approaching or exceeded.
Resource Management Policies: Enforce policies such as auto-scaling, reserved instance usage, and cost optimization best practices.

#### Closing Summary

By setting up a structured showback/chargeback system, establishing clear performance targets through a commitment strategy, and implementing robust guardrails, we can achieve the desired 25% cloud spend reduction while maintaining reliability. The process involves continuous monitoring, feedback loops, and iterative improvements based on actual usage patterns.

#### Common Mistakes/Follow-Ups
Over-Engineering: Teams might implement overly complex financial models that are hard to maintain.
Ignoring Reliability: Cost-cutting measures could compromise critical services if not balanced with reliability targets.
Lack of Feedback Loops: Without continuous monitoring and adjustments, the model may drift away from its goals.

**Comparison:**

```
Subscription	Budget Name	Threshold Value (USD)	Threshold Percentage
subscription1	DevOpsBudgetQ2	5000	80
subscription2	ProductivityBudgetQ1	7500	90
```

**Code:**

```python
# Example Python script for setting up Azure budgets using Azure SDK
from azure.mgmt.resource import ResourceManagementClient
from azure.common.client_factory import get_client_from_auth_file

def set_budget(subscription_id):
    client = get_client_from_auth_file(ResourceManagementClient)

    # Define budget thresholds
    threshold_value = 10000  # Example: $10,000 budget limit

    budget_body = {
        "properties": {
            "timeGrain": "P1D",
            "timePeriod": ["2023-06", "2024-05"],
            "threshold": threshold_value,
            "thresholdPercentage": 80,  # Example: 80% of budget used
            "unit": "Currency"
        }
    }

    client.budgets.create_or_update(subscription_id, 'example-budget', budget_body)
```

**Example input / output:**

```text
Input: Subscription budget thresholds set for Q2.
Output: Notifications sent to teams when budgets are approaching $8,000 or more.
```

**Data:**

```json
{
  "subscriptions": [
    {
      "id": "subscription1",
      "budgets": [
        {
          "name": "DevOpsBudgetQ2",
          "thresholdValue": 5000,
          "thresholdPercentage": 80
        }
      ]
    },
    {
      "id": "subscription2",
      "budgets": [
        {
          "name": "ProductivityBudgetQ1",
          "thresholdValue": 7500,
          "thresholdPercentage": 90
        }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Budget Exceeded?
5. Send Notification
6. Monitor and Adjust
7. Adjust Budget or NotifyStakeholders
8. Continue Monitoring

