---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd govern the cost of a real-time analytics store (Pinot, Druid, ClickHouse) where segment replication, retention, and always-hot memory drive spend, versus serving the same queries from a cheaper batch warehouse with higher latency. Where do you draw the freshness-versus-cost line per query class?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the cost of a real-time analytics store like Pinot, Druid, or ClickHouse while minimizing spend and optimizing query performance, we need to carefully balance several factors including segment replication, retention policies, and memory usage against query latency requirements. We'll also compare this with serving queries from a more cost-effective batch warehouse that has higher latency but can handle less frequent, aggregate reporting needs.

The core challenge is to determine the optimal trade-offs between freshness (query latencies) and cost by defining different query classes and their respective cost thresholds. This involves understanding the query patterns, data access patterns, and business priorities for each class of queries.

#### Triage
What the Question is Really Asking
Question: How can we govern costs in a real-time analytics store while still providing timely results? How do we balance this with serving the same queries from a cheaper batch warehouse?
What's Being Probed:
Understanding of FinOps principles and cloud cost governance.
Knowledge of real-time vs. batch analytics trade-offs.
Ability to design flexible, scalable solutions that optimize both cost and performance.

#### Assumptions
The real-time store (Pinot, Druid, ClickHouse) is mission-critical for timely insights but incurs higher costs due to segment replication, retention policies, and memory usage.
A batch warehouse can be used for aggregate reporting with lower latency requirements, offering a more cost-effective solution.
Step-by-Step Solution

Identify Query Classes:

Define different query classes based on their purpose (e.g., ad-hoc exploration, periodic reports, critical alerts).

Determine Freshness Requirements:

For each query class, define the acceptable freshness level (latency) required.

#### Example
Ad-hoc exploratory queries might require sub-second latency, while monthly reporting could tolerate 15-minute latencies.

Cost Modeling:

Establish a cost model for real-time stores, considering factors like segment replication factor, retention period, and memory usage.
For the batch warehouse, consider the cost of storage and processing capacity at different time scales (e.g., daily snapshots).

Dynamic Cost Thresholds:

Implement a dynamic cost threshold system that adjusts based on query class and real-time store costs.

#### Example
If a critical alert requires immediate insight, prioritize it by writing the data to an in-memory cache or using segment replication.

Cost Optimization Strategies:

Use techniques like tiered storage (HOT/LABS) in Druid for different levels of freshness and cost.
Implement query caching to reduce redundant queries on the real-time store.
Employ schema-on-read strategies where data is read into a more optimized format before analysis.

Monitoring and Alerts:

Set up monitoring to track query performance, costs, and potential bottlenecks.
Establish alerts for cost overruns or suboptimal query execution times.

Automated Cost Management:

Leverage automated tools like AWS Cost Explorer, Google Cloud Pricing Calculator, or Azure Advisor to manage and optimize costs dynamically.
Use machine learning models to predict future costs based on historical data.

#### Closing Summary

By defining clear query classes and their associated freshness requirements, we can effectively govern the cost of a real-time analytics store while ensuring timely insights. This involves balancing high-cost, low-latency real-time stores with more cost-effective batch warehouses for periodic reporting. The key is to automate cost management strategies and continuously optimize based on monitoring feedback.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the need for dynamic cost thresholds that can change based on query type.

#### Follow-Up
How would you handle cases where critical insights require low-latency but come with high costs? Would you use an in-memory cache or segment replication, and what are the trade-offs?

**Comparison:**

```
Query Class	Latency Requirement	Cost Threshold
High Latency (Critical)	< 5 seconds	High
Low Latency (Reporting)	10-30 minutes	Medium
Standard	> 30 minutes	Low
```

**Code:**

```python
# Example of a cost model function
def calculate_cost(queries, segments):
    total_cost = 0
    for query, segs in queries.items():
        if "critical" in query:
            # Higher cost for critical insights
            total_cost += len(segs) * HIGH_COST_FACTOR
        else:
            total_cost += len(segs)
    return total_cost

# Example of a query classification function
def classify_queries(queries):
    classified = {}
    for q in queries:
        if "critical" in q:
            classified["high_latency"] = [q]
        elif "reporting" in q:
            classified["low_latency"] = [q]
        else:
            classified["standard"] = [q]
    return classified
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "queries": [
    {"id": "Q1", "type": "critical"},
    {"id": "Q2", "type": "reporting"},
    {"id": "Q3", "type": "standard"}
  ]
}

Output:

PYTHON
Copy
classified_queries = classify_queries(queries)
print(classified_queries)

# Output: {'high_latency': ['Q1'], 'low_latency': ['Q2'], 'standard': ['Q3']}
```

**Data:**

```json
{
  "queries": [
    {"id": "Q1", "type": "critical"},
    {"id": "Q2", "type": "reporting"},
    {"id": "Q3", "type": "standard"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Query Classes
6. Are Queries Critical?
7. Use Real-Time Store for HighLatency
8. Is Query Periodic Reporting?
9. Write to Batch Warehouse -Low Latency, Medium Cost
10. Use Real-Time Store withStandard Latency
11. Maintain In-Memory Cache forSub-second Latencies
12. Migrate Data to Hot/LABS Tierin Druid
13. Optimize Query Execution andUse Schema-On-Read

