---
type: "practice"
difficulty: "Deep"
---

# Your CloudWatch/Datadog bill is dominated by high-cardinality custom metrics. How do you find the offending dimensions and reduce cardinality without losing debuggability?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a cloud billing issue where high-cardinality custom metrics are driving up costs, we need to identify the dimensions that contribute most to this high cardinality. The goal is to reduce the number of distinct metric data points (dimensions) without compromising on debuggability or critical insights. Here’s how we can approach this:

Understand the Problem and Setup: High-cardinality metrics often arise from too many unique combinations of dimension values, leading to excessive storage costs in monitoring tools like CloudWatch and Datadog.

#### Assumptions
and Scope: We assume that the custom metrics are used for tracking key performance indicators (KPIs) or application health. Our objective is to minimize dimensions while ensuring we can still effectively debug issues when needed.
Step-by-Step Solution

Identify Metrics with High Cardinality:

Use CloudWatch’s or Datadog's metric explorer to filter and inspect metrics that have a high cardinality.
Look for metrics where the number of distinct values (dimensions) is disproportionately large compared to other similar metrics.

Analyze Dimension Combinations:

List out all dimensions associated with each offending metric.
Analyze which combinations are most common versus those that occur infrequently but still provide critical insights.

Prioritize and Group Dimensions:

Identify key business or technical dimensions that should always be tracked (e.g., environment, service name).

#### Cons
ider grouping secondary dimensions into broader categories to reduce cardinality.
For example, instead of tracking individual user IDs, group them by role or department.

Impact Analysis:

Perform a risk assessment for each potential reduction in dimensionality.
Ensure that any changes won't compromise the ability to debug issues effectively.

#### Implementation
and Monitoring:

Implement the reduced set of dimensions using CloudWatch's custom metrics configuration or Datadog’s tagging rules.
Monitor the impact on storage costs and debugging capabilities over a period.
Gradually roll out changes if necessary, based on feedback from operations teams.

#### Closing Summary

By methodically analyzing high-cardinality metrics and strategically reducing unnecessary dimensions, we can significantly lower cloud monitoring costs without sacrificing debuggability. Key steps include identifying problematic metrics, analyzing dimension combinations, prioritizing critical dimensions, performing a risk analysis, and implementing changes with ongoing monitoring.

#### Common Mistakes/Follow-Ups

Over-reducing Dimensions: Avoid removing too many dimensions that are essential for debugging.
Ignoring Historical Data: Ensure any change does not disrupt historical data or analysis.
Lack of Feedback Mechanism: Establish a process to gather feedback from operations and development teams post-implementation.

**Code:**

```python
# Example Python code snippet to filter high-cardinality metrics using CloudWatch API
import boto3

cloudwatch = boto3.client('cloudwatch')

response = cloudwatch.list_metrics()
high_cardinality_metrics = []
for metric in response['Metrics']:
    dimensions = len(metric['Dimensions'])
    if dimensions > 10:  # Threshold for high cardinality, adjust as needed
        high_cardinality_metrics.append(metric)

print(high_cardinality_metrics)
```

**Example input / output:**

```text
Input:
- CloudWatch metric with dimensions [Environment, ServiceName, Role, User_ID]

Output:
- Reduced dimensionality metric with dimensions [Environment, ServiceName, Role] (User_IDs are aggregated by department/role)
```

**Data:**

```yaml
# Example of a custom CloudWatch metric configuration
dimensions:
  - Name: Environment
    Value: Prod
  - Name: ServiceName
    Value: MyApp
  - Name: Role
    Value: Admin
  - Name: User_IDs # This could be grouped into Department or Role
```

