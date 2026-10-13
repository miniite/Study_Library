---
type: "practice"
difficulty: "Deep"
---

# Design budget pacing for an ad campaign so a daily budget is spent smoothly across the day without overspend — how do you distribute the budget across serving nodes and reconcile spend in near-real-time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level architecture for distributing and reconciling an ad campaign's daily budget, we need to ensure that the spending is smooth and consistent throughout the day while avoiding overspend. At this level of detail, we're focusing on the overall system design rather than implementation specifics.

The core idea here is to distribute the budget evenly across time slots or serving nodes and continuously monitor and adjust spending in near-real-time. This involves breaking down the daily budget into smaller increments, tracking spend per increment, and dynamically reallocating funds as needed based on current performance metrics.

First, let's break down the problem:

Budget Distribution: Divide the total daily budget into smaller intervals (e.g., minutes).
Serving Nodes: Implement nodes responsible for serving ads within each interval.
Real-time Monitoring and Reconciliation: Continuously track spend per node and adjust as necessary to ensure even distribution.

We'll reason through this from first principles:

Define the time slots: For a 24-hour campaign, we can divide it into intervals like 5-minute or 10-minute periods.
Assign budget to each slot: Distribute the daily budget evenly across these slots.
Implement serving nodes: Create nodes that handle ad requests within their assigned time slots.
Monitor spend: Use a monitoring system to track actual spending in real-time.
Reallocate budget: If a node is overspending, redistribute funds from other underperforming nodes.

To summarize the approach:

Time Slots: Divide 24 hours into smaller intervals (e.g., 10 minutes).
Budget Allocation: Allocate an equal portion of the daily budget to each interval.
Serving Nodes: Deploy nodes responsible for serving ads during their assigned time slots.
Real-time Monitoring: Use a backend system to monitor spend per node in real-time.
Reconciliation and Reallocation: Adjust spending dynamically based on observed performance.

Key Challenges:

Ensuring Even Distribution: The budget must be evenly spread across intervals without manual intervention.
Near-real-time Reconciliation: Timely updates are crucial to prevent overspend.
Performance Metrics: Define what constitutes "good" performance for each node to make informed reallocation decisions.

**Comparison:**

```
Interval	Budget Allocated	Actual Spend
0	$69.44	$72.58
1	$69.44	$68.81
...	...	...
143	$69.44	$71.32
Interval	Allocated vs Actual Spend Difference
0	+$3.14
1	-$0.63
...	...
143	+$1.88
```

**Code:**

```python
# Example Python code snippet for budget distribution and reconciliation

def distribute_budget(daily_budget, num_intervals):
    interval_budget = daily_budget / num_intervals
    return {f'interval_{i}': interval_budget for i in range(num_intervals)}

def monitor_and_reconcile(node_stats, current_budget, threshold=0.95):
    underperforming_nodes = [node for node, spend in node_stats.items() if spend / budget < threshold]
    overperforming_nodes = {node: spend for node, spend in node_stats.items() if spend > interval_budget * 1.2}

    # Reallocate funds
    total_reallocated = sum(overperforming_nodes.values())
    for node in underperforming_nodes:
        reallocated_amount = min(overperforming_nodes[node], threshold * budget)
        overperforming_nodes[node] -= reallocated_amount
        node_stats[node] += reallocated_amount

    return node_stats, total_reallocated

# Example usage
budget_distribution = distribute_budget(daily_budget=10000, num_intervals=144)  # 24 hours * 60 minutes / interval
node_monitoring = {f'interval_{i}': random.uniform(0.85, 1.15) * budget_distribution[f'interval_{i}'] for i in range(len(budget_distribution))}
rebalanced_stats, reallocated_funds = monitor_and_reconcile(node_monitoring, current_budget=2000)
```

**Example input / output:**

```text
Input:
- Daily budget: $10,000
- Number of intervals: 144 (24 hours * 60 minutes / interval)

Output:
- Budget distribution per interval: $69.44 each
- Node monitoring data:
    - Interval 0: $72.58 spent
    - Interval 1: $68.81 spent
    - ...
    - Interval 143: $71.32 spent

Rebalanced stats after reconciliation:
- Underperforming nodes identified and funds reallocated to overperforming ones.
```

**Data:**

```json
{
    "budget_distribution": {
        "interval_0": 69.44,
        "interval_1": 69.44,
        ...
        "interval_143": 69.44
    },
    "node_monitoring": {
        "interval_0": 72.58,
        "interval_1": 68.81,
        ...
        "interval_143": 71.32
    }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Start
5. Is Daily Budget Exceeded?
6. Distribute budget evenly
7. Monitor spend in real-time
8. Is Spend Balanced AcrossNodes?
9. End
10. Reallocate funds fromoverperforming tounderperforming nodes
11. Maintain balanced distribution

