---
type: "practice"
difficulty: "Deep"
---

# Your data-warehouse compute is on-demand credits and a single unreviewed dbt model change 10x'd the daily cost overnight. Design the cost-preview-in-PR gate for warehouse workloads and explain what you can estimate before the query runs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Query Cost Guardrails](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Query_Cost_Guardrails.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a FinOps and cloud cost governance context, the goal is to ensure that changes to data pipelines or queries do not result in unexpected costs. For on-demand credits-based compute environments like Amazon Redshift or Google BigQuery, implementing a cost-preview-in-PR (Pull Request) gate can be crucial. This means reviewing the estimated cost of any database transformation tool (dbt model) change before it is merged into production.

#### Triage

The core question here involves designing an automated mechanism that estimates and previews the costs associated with running dbt models, specifically in a cloud data-warehouse setting. The key aspect is to gate these changes based on predefined cost thresholds to prevent overspending. The interviewer might be probing your understanding of cost optimization techniques, dbt best practices, and how to integrate such checks into the software development lifecycle.

#### Assumptions
Cost Model: We will use the cloud provider’s pricing model (e.g., AWS Redshift or Google BigQuery).
dbt Model: The changes are in the form of SQL transformations.
Automation Tools: Existing CI/CD pipelines and cost estimation APIs.

#### First-Principles Solution
Step 1: Define Cost Estimation Logic

The first step is to understand how each dbt model change will be translated into a cost estimate. This involves:

Parsing the SQL of the dbt models.
Understanding which tables or partitions are involved.
Calculating the expected execution time and resource usage.
Step 2: Integrate Cost Estimation with CI/CD

We need to integrate this logic into our existing CI/CD pipeline, ensuring that every PR triggers a cost estimation. This can be achieved by:

Parsing dbt Models: Use dbt’s output directory to parse the SQL files.
Resource Calculation: Calculate the expected resource usage for each query.
Cost Estimation API: Utilize the cloud provider's APIs (e.g., AWS Cost Explorer, Google Cloud Billing API) to estimate costs based on resource usage.
Step 3: Automated Gate Mechanism

The automated mechanism should:

Run cost estimates during the CI/CD pipeline stages.
Compare the estimated cost against predefined thresholds.
Block PRs that exceed these thresholds unless approved by a cost guardian or finance team member.

#### Implementation
Example
Yes
No
Parse dbt Model Changes
Calculate Resource Usage
Estimate Costs using CloudAPIs
Evaluate Against Thresholds
Cost Below Threshold?
Approve PR
Notify Developer/FinanceTeam

#### Closing Summary

In summary, the solution involves integrating cost estimation into the CI/CD pipeline to ensure that any dbt model change is reviewed for its potential impact on costs. This requires parsing dbt models, calculating resource usage, and utilizing cloud provider APIs to estimate costs.

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to consider edge cases where cost estimation might be inaccurate.

#### Follow-Up
Discussing how to handle complex queries that span multiple databases or require joins between tables from different data sources.
Mistake 2: Overlooking the need for manual overrides in exceptional circumstances.

#### Follow-Up
Explaining mechanisms for finance teams to override cost gates and the processes around it.
Final Thoughts

Cost governance is critical in cloud environments, especially when using on-demand credits. By automating this process, we can significantly reduce the risk of unexpected costs while ensuring high-quality transformations are implemented efficiently.

**Comparison:**

```
Resource	Usage Calculation
CPU	Number of Queries * Query Duration
Memory	Data Size processed / Batch Size
```

**Code:**

```python
# Example Python script for parsing dbt models and estimating costs

import boto3
from dbt.utils import parse_dbt_files

def estimate_cost(model_file):
    # Parse the model SQL file
    parsed_sql = parse_dbt_files([model_file])

    # Calculate resource usage (simplified example)
    resource_usage = calculate_resource_usage(parsed_sql)

    # Estimate cost using AWS Cost Explorer API
    client = boto3.client('ce')
    response = client.get_cost_and_usage(
        TimePeriod={
            'Start': '2023-01-01',
            'End': '2023-01-31'
        },
        Granularity='DAILY',
        Metrics=['AmortizedCost'],
        Filter={
            'Dimensions': {
                'Key': 'SERVICE',
                'Values': ['AmazonRedshift']
            }
        }
    )

    estimated_cost = sum([data['Total']['Amount'] for data in response['ResultsByTime']])
    return resource_usage, estimated_cost
```

**Example input / output:**

```text
Input:
- PR with dbt model changes

Output:
- Estimated cost for each model change
- Gate decision (approve or block) based on threshold
```

**Data:**

```json
{
  "model_files": [
    "/path/to/dbt_model_1.sql",
    "/path/to/dbt_model_2.sql"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Parse dbt Models
4. Calculate Usage
5. Estimate Costs
6. Evaluate Against Thresholds
7. Cost Below Threshold?
8. Approve PR
9. Notify Developer/FinanceTeam

