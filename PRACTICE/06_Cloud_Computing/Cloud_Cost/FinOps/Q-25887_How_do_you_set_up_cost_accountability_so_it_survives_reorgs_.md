---
type: "practice"
difficulty: "Deep"
---

# How do you set up cost accountability so it survives reorgs and team boundary changes? Tagging keyed to teams breaks every quarter — design an attribution scheme resilient to organizational churn.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Funding Models and Incentives](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Funding_Models_and_Incentives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure cost accountability remains resilient to organizational churn in a cloud environment, we need to design an attribution scheme that is decoupled from team structures and organizational hierarchies. This involves leveraging dynamic tagging strategies, detailed cost models, and automated reporting tools. By focusing on these elements, we can create a system that accurately tracks costs across multiple teams and projects without relying solely on static tags.

#### Triage
The question revolves around designing a robust cost attribution model for cloud environments. It probes our ability to handle organizational changes, manage dynamic tagging, and ensure accurate cost distribution even when team boundaries shift. We must make assumptions about the current state of the organization and the types of changes we expect. For this solution, let's assume a multi-team environment with frequent reorganizations.

Step-by-step Solution:

Dynamic Tagging: Use AWS Cost Explorer or Azure Cost Management to create dynamic tags that capture resource usage in real-time. This involves tagging resources based on logical identifiers rather than static team names.
Cost Allocation Rules: Implement cost allocation rules within cloud service providers (CSP) such as AWS, Azure, or GCP to automatically assign costs to the appropriate projects and departments.
Granular Cost Models: Develop detailed cost models that consider various dimensions like application, environment, project, and department. This helps in distributing costs accurately even when team structures change.
Automated Reporting: Use tools like Datadog or New Relic to generate automated reports that provide insights into cloud spending by different teams and projects.

#### Closing Summary
By adopting a dynamic tagging strategy and leveraging cost allocation rules, we can ensure that cost accountability remains robust even as organizational boundaries shift. This approach not only simplifies the process of adjusting to reorgs but also provides deeper insights into how resources are being used across the organization.

#### Common Mistakes/Follow-Ups

Relying too heavily on static tagging and team names.
Ignoring the need for granular cost models that can accommodate changing organizational structures.
Failing to automate reporting, leading to manual overhead during reorganizations.
Not considering the integration of third-party tools like Datadog or New Relic.

**Comparison:**

```
Tag Key	Value Type
Application	String
Environment	String
Project	String
Department	String
```

**Code:**

```python
# Example Python script for dynamic tagging in AWS using Boto3
import boto3

def tag_resource(resource_id, tag_key, tag_value):
    client = boto3.client('ec2')
    response = client.create_tags(
        Resources=[resource_id],
        Tags=[
            {'Key': tag_key, 'Value': tag_value}
        ]
    )
    return response
```

**Example input / output:**

```text
Input:
Resource ID: r-1234567890abcdef
Tag Key: Application
Tag Value: App-Team

Output:
{
    "ResponseMetadata": {
        "RequestId": "abcde-fghij-klmno-pqrst",
        "HTTPStatusCode": 200,
        "HTTPHeaders": {
            "date": "Mon, 25 Jul 2022 16:39:48 GMT",
            "content-type": "application/json",
            "content-length": "96"
        },
        "RetryAttempts": 0
    }
}
```

**Data:**

```json
{
  "tag_keys": [
    {"key": "Application", "value_type": "String"},
    {"key": "Environment", "value_type": "String"},
    {"key": "Project", "value_type": "String"},
    {"key": "Department", "value_type": "String"}
  ],
  "cost_models": {
    "app1": {"Application": "App-Team", "Environment": "Dev", "Project": "P-001", "Department": "IT"},
    "app2": {"Application": "App-Team", "Environment": "Prod", "Project": "P-002", "Department": "Sales"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Resource Creation
4. Is Dynamic Tagging Required?
5. Tag Resource with LogicalIdentifier
6. Use Static Tags if Possible
7. Monitor and Adjust CostModels as Needed
8. Update Static TagsOccasionally
9. Generate Automated Reportsfor Cost Analysis

