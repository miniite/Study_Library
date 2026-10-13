---
type: "practice"
difficulty: "Deep"
---

# A team keeps hitting their budget cap and quietly moves workloads to a personal AWS account outside governance. How do you architect the org and detection so shadow-IT spend can't escape your FinOps net?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow IT and Budget Evasion](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_IT_and_Budget_Evasion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to build an architecture that ensures all cloud resources are under centralized governance while also detecting and preventing unauthorized usage of personal AWS accounts. The core idea is to enforce a unified cloud cost strategy and create robust monitoring and alerting mechanisms.

First, let's clarify what the question is really asking: We must design a system that prevents shadow IT (unauthorized use of cloud services) from bypassing our FinOps controls. This involves setting up policies, integrating tools, and establishing processes to ensure all resources are under centralized management and any deviation from this is detected and remediated.

Key Assumptions:
The organization already has a central AWS account for managing costs.
We need to prevent users from creating personal accounts or moving existing resources out of the centralized control.
Our system should detect and alert us when such activities occur.
There is an existing budget cap mechanism in place, but it's being circumvented.

#### Solution

Centralized Management with Policy Enforcement:

Use AWS Organizations to create a hierarchical structure that enforces governance policies on all accounts under management.
Implement IAM policies and resource tagging across the organization to ensure only authorized users can perform specific actions.

Unified Cost Allocation Tagging Strategy:

Introduce a uniform tagging strategy for all resources deployed in the central account. This ensures traceability and helps in accurate cost allocation.
Use AWS Budgets to define spending limits per tag or service and automatically trigger alerts when thresholds are breached.

Continuous Monitoring and Detection:

Implement CloudTrail to log API calls and ensure all changes made to resources are auditable.
Use AWS Config for continuous assessment of resource configurations against the established policies, generating notifications on any non-compliance.

Automated Remediation:

Develop a script or use an automation service like AWS Lambda triggered by CloudTrail events to automatically revert unauthorized actions (e.g., deleting rogue accounts or moving resources back).
Integrate with AWS Service Catalog to ensure only approved services can be launched within the organization’s cloud environment.

Enhanced Visibility and Reporting:

Deploy AWS Trusted Advisor to provide ongoing recommendations for cost optimization and security best practices.
Utilize AWS Cost Explorer and Amazon QuickSight to create dashboards that provide real-time visibility into spending patterns and trends.

Security and Compliance Audits:

Conduct regular audits using AWS Config Rules and Security Hub to ensure compliance with organizational policies.
Implement a culture of security awareness training for all employees to prevent accidental violations.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, the architecture should be designed to enforce strict governance through centralized management, unified tagging strategies, continuous monitoring, automated remediation, enhanced visibility, and regular audits. Key pitfalls include neglecting to implement a strong security culture or underestimating the complexity of managing multiple accounts effectively.

Common follow-up questions might include how to handle different departments with varying cloud usage requirements, integrating third-party tools for cost management, or dealing with edge cases where legitimate business needs might overlap with shadow IT activities.

**Comparison:**

```
Resource	Tag Key	Tag Value
S3 Bucket	Owner	Finance
EC2 Instance	Purpose	DevOps
Action	CloudTrail Event Name	Description
Move Resources	CreateStack, DeleteStack	Indicates unauthorized resource movement.
```

**Code:**

```python
# Example Python script for automated remediation using AWS Lambda and CloudTrail
import boto3

def lambda_handler(event, context):
    client = boto3.client('cloudtrail')
    # Check for specific API calls indicating unauthorized resource movement
    response = client.lookup_events(
        LookupAttributes=[
            {
                'AttributeKey': 'EventName',
                'AttributeValue': 'MoveResources'
            }
        ],
        StartTime=datetime.now() - timedelta(days=1)
    )

    if response['Events']:
        # Remediate the detected action (e.g., delete rogue account or move resources back)
        for event in response['Events']:
            print(f"Detected unauthorized action: {event['EventName']}")
            # Implement logic to automatically revert actions
```

**Example input / output:**

```text
Input:
- User attempts to move a resource from the central account to a personal AWS account.

Output:
- CloudTrail logs API call indicating unauthorized action.
- AWS Config triggers an alert for non-compliance.
- Lambda function automatically reverts the unauthorized action by moving the resource back to the central account and sends an email notification.
```

**Data:**

```json
{
  "OrganizationsSettings": {
    "AllAwsAccountsEnabled": true,
    "MasterAccountRoleArn": "arn:aws:iam::123456789012:role/ServiceRole",
    "DefaultPolicyType": "CONTROL"
  },
  "Budgets": [
    {
      "BudgetName": "DailyCostLimit",
      "BudgetedAmount": 500,
      "ThresholdPercentage": 90
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. CloudTrail Logs API Call
4. Is Unauthorized Action?
5. Lambda Function RevertsAction
6. AWS Config Triggers Alert
7. Monitor Next Event

