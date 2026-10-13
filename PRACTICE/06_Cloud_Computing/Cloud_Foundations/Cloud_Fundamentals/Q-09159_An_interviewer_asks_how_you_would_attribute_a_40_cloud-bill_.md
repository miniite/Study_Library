---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer asks how you would attribute a 40% cloud-bill jump to a specific team when everything sits in one shared account. What tagging and cost-tooling approach would you set up?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

To address the issue of attributing a significant increase in cloud billing—specifically a 40% jump—to a specific team within a shared account, we need to implement detailed tagging and cost analysis tools. This ensures that each team can be billed according to their actual usage, fostering cost-conscious behavior and enabling better financial planning.

#### Triaging the Question

The interviewer is probing how you would implement a robust tagging strategy and utilize AWS Cost Explorer or similar tools to track and analyze costs. We need to ensure that resources are tagged appropriately so that they can be associated with different teams, and then use these tags in cost analysis reports to identify which team's activities contributed most to the increased spending.

#### First-Principles Solution

Tagging Strategy:

Define a consistent tagging scheme for all cloud resources.
Use meaningful tag keys like team-name, project-code, and environment.
Ensure that resource owners or administrators are responsible for applying these tags correctly during deployment.

Implementing Tags in AWS:

For EC2 instances, ensure tags like team-name are applied.
For S3 buckets, use tags such as project-code to categorize the data stored.
Use IAM policies and roles to enforce tagging practices, ensuring that untagged resources cannot be deployed or modified.

Cost Analysis with AWS Cost Explorer:

Utilize AWS Cost Explorer to create detailed cost reports based on tags.
Set up cost budgets and alerts to monitor deviations from expected costs.
Regularly review the cost data to identify any anomalies that may indicate excessive spending.

Automation and Monitoring:

Use CloudWatch alarms or EventBridge rules to trigger notifications for untagged resources or cost anomalies.
Implement automation scripts using AWS Lambda to enforce tagging policies if necessary.

#### Closing Summary

By setting up a robust tagging strategy and leveraging cost-analysis tools, we can accurately attribute cloud costs to specific teams. This approach not only helps in identifying cost drivers but also promotes better resource management practices within the organization.

#### Common Mistakes/Follow-Ups
Not Enforcing Tagging Policies: Ensuring that all resources are consistently tagged is crucial; otherwise, it becomes difficult to track costs accurately.
Ignoring Cost Budgets and Alerts: Setting up cost budgets can help prevent unexpected charges but must be followed by proactive monitoring and action.
Lack of Regular Reviews: Continuous review of cost data and usage patterns ensures that any unusual spending trends are promptly addressed.

If asked for a specific example, I might use AWS Cost Explorer to create a breakdown based on team-name tags. For instance:

TEXT
Team A: 35% of total costs
Team B: 20%
Team C: 18%

This would highlight that Team A is the main contributor and should be closely monitored.

**Comparison:**

| Team Name   | Percentage of Total Costs |
|-------------|--------------------------|
| DevOps      | 35%                      |
| Product A   | 20%                      |
| Product B   | 18%                      |

**Code:**

```python
# Example Lambda function to enforce tagging
def lambda_handler(event, context):
    if not event.get('tags'):
        raise ValueError("Resource must have tags applied")
    return "Tags applied successfully"
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Create an S3 bucket with tags `team-name=DevOps` and `project-code=P12345` | Bucket created successfully, tagged properly. |

**Data:**

```json
{
  "Tags": {
    "team-name": "DevOps",
    "project-code": "P12345"
  }
}
```

**Flow:**

1. Yes
2. No
3. Create Resource
4. Apply Tags?
5. Resource Created with Tags
6. Alert Unsuccessful Tagging,Enforce Policy
7. Categorize Costs in AWS CostExplorer
8. Monitor and Correct TaggingPractices

