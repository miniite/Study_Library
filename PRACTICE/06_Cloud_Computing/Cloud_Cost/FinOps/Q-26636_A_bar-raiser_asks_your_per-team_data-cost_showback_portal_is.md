---
type: "practice"
difficulty: "Deep"
---

# A bar-raiser asks: your per-team data-cost showback portal is technically accurate but teams ignore it. What behavioral-design and workflow-integration choices — surfacing cost inside the query editor, weekly deltas in Slack, cost in the dbt run summary — actually change data-engineer behavior versus another ignored dashboard?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of teams ignoring a per-team data-cost showback portal, we need to understand that behavioral design and workflow integration are key in driving adoption. The goal is not just to provide accurate cost information but to make it an integral part of the engineers' daily activities, thereby ensuring they are continuously aware of their costs.

#### Triaging the Question

The question is asking about how different surface-level changes can influence data-engineer behavior regarding cloud cost governance. Specifically, we need to explore:

Surfacing Cost in the Query Editor: How visibility and immediate feedback during query execution can impact behavior.
Weekly Deltas in Slack: The role of real-time notifications on cost deltas for fostering accountability.
Cost in dbt Run Summary: Embedding cost metrics directly into the workflow to ensure continuous awareness.

#### Assumptions
Teams are technically capable but may not be aware or motivated by traditional dashboards.
Behavioral changes can be driven through integration with tools already used during daily operations (like query editors, Slack, and dbt).

#### First-Principles Solution
Surfacing Cost in the Query Editor

Embedding cost metrics directly into the SQL query editor can provide immediate feedback. This approach works because:

Real-Time Awareness: Engineers get instant insights on costs as they write or modify queries.
Contextual Relevance: Cost information is immediately relevant to the task at hand, increasing likelihood of adoption.

#### Implementation
Example:

Integrate a cost estimation API that runs alongside the query execution in tools like DBeaver, pgAdmin, or SQL Workbench.
Provide a tooltip or inline message displaying estimated costs for each query component (e.g., tables, joins).
Weekly Deltas in Slack

Sending weekly cost deltas via Slack can drive regular attention to cost management:

Timely Notifications: Real-time notifications ensure that cost increases are noticed before they become problematic.
Social Accountability: Group discussions and reminders can foster a culture of cost awareness.

#### Implementation
Example:

Use AWS Cost Explorer or similar tools to generate weekly reports.
Set up automated Slack integrations to send these reports to relevant teams, including specific members who may be responsible for the costs.
Cost in dbt Run Summary

Embedding cost metrics into the dbt run summary ensures that cost is always top-of-mind during data engineering tasks:

Integrated Experience: Cost becomes part of the end-to-end workflow, reducing cognitive load.
Continuous Monitoring: Real-time updates ensure continuous monitoring and adjustment.

#### Implementation
Example:

Extend the existing dbt hooks to include cost metrics in run summaries.
Integrate with cloud provider APIs to fetch current costs and display them alongside dbt run details.

#### Closing Summary

By surfacing cost information where engineers are already working (query editor, Slack, dbt), we can drive continuous awareness and proactive cost management. Each of these approaches has its strengths:

Query Editor: Immediate feedback during the writing process.
Slack Notifications: Regular reminders to keep costs top-of-mind.
dbt Summary: Ensures cost is always part of the workflow.

#### Common Mistakes/Follow-Ups
Lack of Context: Failing to integrate cost metrics into contexts where engineers are already working can result in poor adoption.
Overcomplexity: Over-engineering solutions that require significant changes to existing workflows can be resistant to implementation.
Misalignment: Ensuring alignment with other tools and processes (like CI/CD pipelines) is crucial for seamless integration.
Triaging Further

Additional strategies could include:

A/B Testing: Gradually roll out different methods to gauge effectiveness.
User Feedback: Regularly solicit feedback from engineers to refine and improve the solution iteratively.

**Comparison:**

```
Query	Estimated Cost
SELECT * FROM sales_data;	0.5
```

**Code:**

```python
# Example of integrating cost metrics into a query editor using DBeaver API
def get_estimated_cost(sql_query):
    # Dummy implementation
    return {"cost": 0.5, "details": "Cost breakdown"}

# Integration with dbt hooks
class CostTrackingHook(dbt.Hook):
    def before_node(self, node):
        estimated_cost = get_estimated_cost(node.compiled_sql)
        print(f"Estimated cost for {node.unique_id}: ${estimated_cost['cost']}")
```

**Example input / output:**

```text
Input:
SELECT * FROM sales_data;

Output:
Estimated cost for <unique_id>: $0.5

Details:
- sales_data: $0.3
- join_sales_customers: $0.2
```

**Data:**

```json
{
  "query": "SELECT * FROM sales_data",
  "estimated_cost": {
    "cost": 0.5,
    "details": [
      {"component": "sales_data", "cost": 0.3},
      {"component": "join_sales_customers", "cost": 0.2}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Writing
4. Cost Estimation
5. Real-Time Feedback
6. Cost Notification
7. Immediate Cost Awareness
8. Slack Notifications

