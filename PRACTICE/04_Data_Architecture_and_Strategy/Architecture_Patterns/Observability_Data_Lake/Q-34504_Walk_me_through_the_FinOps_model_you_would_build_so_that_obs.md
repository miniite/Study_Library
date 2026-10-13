---
type: "practice"
difficulty: "Expert"
---

# Walk me through the FinOps model you would build so that observability-lake spend is allocated back to teams in a way that actually changes their instrumentation behavior.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of allocating observability lake spend back to teams to influence their instrumentation behavior, we need to build a FinOps model that integrates cost tracking with behavioral incentives. This involves several key steps, including defining cost metrics, integrating with observability tools, and implementing an automated allocation system.

#### Triaging the Question

The core of this question is about creating a mechanism for financial transparency and accountability within a company's observability infrastructure. The goal is to ensure that teams are aware of their spending on observability data lakes and incentivize them to optimize their instrumentation practices. This requires understanding how costs are incurred, tracking those costs accurately, and then distributing the costs back to relevant teams in a meaningful way.

#### Assumptions
Cost Tracking: We assume that there is an existing system for tracking cost allocations within the observability infrastructure.
Observability Tools Integration: The observability tools (e.g., Prometheus, Jaeger) are already deployed and gathering data.
Team Structure: Teams are well-defined and have clear ownership over different parts of the application stack.

#### Solution
from First Principles

Define Cost Metrics:

Identify the cost drivers in the observability lake (e.g., storage costs, compute usage, query volumes).
Calculate these costs on a per-team or per-application basis.

Integrate with Observability Tools:

Use existing telemetry data to calculate the costs associated with each team's instrumentation practices.
Implement custom metrics in observability tools that capture the relevant cost information.

Automated Allocation System:

Develop a system that aggregates and allocates these costs back to teams based on their usage patterns.
Ensure that the allocation is transparent and verifiable, using dashboards and reports.

Behavioral Incentives:

Design financial incentives for teams to reduce unnecessary instrumentation or optimize their practices.
Provide real-time cost alerts and insights to help teams make informed decisions.

Feedback Loops:

Create a feedback loop where teams can see the impact of their instrumentation choices on costs.
Use these insights to drive continuous improvement in their observability strategies.

#### Closing Summary

By integrating cost tracking with behavioral incentives, we can create a FinOps model that not only tracks spending but also influences how teams approach their observability practices. This involves defining clear metrics, automating the allocation of costs back to relevant teams, and providing actionable insights through dashboards and reports.

#### Common Mistakes/Follow-Ups
Overlooking Cost Drivers: Failing to identify all cost drivers can result in an incomplete picture.
Lack of Transparency: Without transparent tracking and reporting, teams may not understand the impact of their decisions.
Inadequate Incentives: Poorly designed incentives might fail to motivate teams effectively.

#### Implementation
Complexity: The complexity of integrating with existing observability tools can be underestimated.

**Comparison:**

```
Team	Cost Metric	Storage Cost	Compute Cost	Queries Cost	Total Cost
A	storage	100	-	-	10.0
A	compute	-	50	-	20.0
A	queries	-	-	6.0	6.0
B	storage	75	-	-	7.5
B	compute	-	30	-	12.0
B	queries	-	-	3.0	3.0
Total Cost for Team A:	39.5
Total Cost for Team B:	18.0
```

**Code:**

```python
# Example Python code for aggregating costs
def calculate_team_costs(usage_data, team_mapping):
    cost_metrics = {
        "storage": 0.1,
        "compute": 0.2,
        "queries": 0.3
    }

    total_cost = {}
    for usage in usage_data:
        team_id = team_mapping[usage["application"]]
        cost = (
            usage["storage"] * cost_metrics["storage"] +
            usage["compute"] * cost_metrics["compute"] +
            usage["queries"] * cost_metrics["queries"]
        )
        if team_id not in total_cost:
            total_cost[team_id] = 0
        total_cost[team_id] += cost

    return total_cost

# Example data schema
usage_data = [
    {"application": "app1", "storage": 100, "compute": 50, "queries": 20},
    {"application": "app2", "storage": 75, "compute": 30, "queries": 10}
]
team_mapping = {
    "app1": "teamA",
    "app2": "teamB"
}

# Calculate costs
costs = calculate_team_costs(usage_data, team_mapping)
print(costs)  # Output: {'teamA': 39.5, 'teamB': 18.0}
```

**Example input / output:**

```text
Input:
- team_mapping = {'app1': 'teamA', 'app2': 'teamB'}
- usage_data = [
    {"application": "app1", "storage": 100, "compute": 50, "queries": 20},
    {"application": "app2", "storage": 75, "compute": 30, "queries": 10}
]

Output:
- {'teamA': 39.5, 'teamB': 18.0}

Explanation:
Team A's cost = (100 * 0.1) + (50 * 0.2) + (20 * 0.3) = 39.5
Team B's cost = (75 * 0.1) + (30 * 0.2) + (10 * 0.3) = 18.0
```

**Data:**

```json
{
    "team_mapping": {
        "app1": "teamA",
        "app2": "teamB"
    },
    "usage_data": [
        {"application": "app1", "storage": 100, "compute": 50, "queries": 20},
        {"application": "app2", "storage": 75, "compute": 30, "queries": 10}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Usage Data Collection
4. Is the data complete?
5. Calculate Costs
6. Data Collection Continues
7. Allocate Costs to Teams
8. Generate Financial Insights &Alerts
9. Distribute Costs to Teams forIncentives

