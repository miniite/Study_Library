---
type: "practice"
difficulty: "Deep"
---

# A single unpartitioned full-table scan in your Trino cluster reads 400TB and blows the daily compute budget. Architect the partition-pruning, statistics, and query-cost-preview governance that makes teams pay only for the data their queries actually need without gatekeeping ad-hoc analysis.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Query Cost Guardrails](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Query_Cost_Guardrails.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When a single unpartitioned full-table scan in Trino consumes 400TB and exceeds daily compute budgets, it highlights the need for robust partitioning strategies, efficient statistics gathering, and comprehensive query cost previews. Our goal is to ensure that teams only pay for the data they actually access, fostering a culture of ad-hoc analysis without compromising financial constraints.

#### Triaging the Question

The question revolves around several key aspects:

Partition Pruning: Ensuring queries can leverage table partitions to reduce scan volumes.
Statistics Collection: Gathering and maintaining accurate statistics about table distributions to improve query optimization.
Query Cost Preview: Providing real-time insights into query costs before execution, enabling informed query optimization.

#### First-Principles Solution
Partition Pruning
Partition Design: Implement a partitioning strategy that aligns with common query patterns. For instance, date-based partitions can be effective for time-series data.
Partition Column Selection: Choose columns that frequently appear in WHERE clauses as partition keys.
Dynamic Partitions: Use dynamic partition pruning to handle varying ranges of partition values.
Statistics Collection
Histograms and Skew: Collect histograms for skewed distributions to improve cardinality estimates.
Regular Refreshes: Ensure statistics are regularly refreshed based on data changes to maintain accuracy.
Query-Driven Stats: Implement query-driven statistics where Trino can gather stats during the execution of queries.
Query Cost Preview
Cost Estimation API: Integrate a cost estimation API that provides estimated costs for queries before they run.
Budget Allocation: Allocate budget limits per team or project to prevent overspending.
Real-Time Alerts: Set up real-time alerts when query costs exceed predefined thresholds.

#### Implementation
Steps

Partitioning Strategy:

Design Phase: Analyze historical and expected query patterns to identify the most frequently used filters.

#### Implementation
Modify table definitions in Trino to include partition columns. Use dynamic partition pruning for queries that span multiple partitions.

Statistics Gathering:

Histograms: Configure Trino to collect histograms on key columns, especially those with skewed distributions.
Refresh Policy: Implement a scheduled task to refresh statistics at regular intervals (e.g., daily or weekly).
Query-Driven Stats: Enable query-driven stats collection for critical tables.

Cost Estimation and Budgeting:

API Development: Develop an API that integrates with Trino’s metadata and cost metrics.
Budget Allocation: Define budget limits per team or project in a centralized management system.
Real-Time Alerts: Integrate real-time alerting mechanisms to notify teams when they approach their budget limits.

#### Closing Summary

By implementing partition pruning, maintaining accurate statistics, and providing real-time query cost previews, we can ensure that data teams pay only for the data they actually need. This approach balances the flexibility of ad-hoc analysis with strict financial controls, fostering a culture of efficient resource utilization.

#### Common Mistakes/Follow-Ups
Ignoring Partition Design: Not considering the underlying query patterns when designing partitions can lead to suboptimal performance.
Inadequate Statistics Maintenance: Failing to regularly refresh and maintain statistics can result in inaccurate cost estimates.
Over-reliance on Budget Limits: Rigid budget limits without real-time monitoring and feedback can stifle ad-hoc analysis.

For follow-ups, it’s important to discuss how these strategies scale with increasing data volumes and query complexity, as well as any trade-offs involved in each approach.

**Comparison:**

| Column   | Type        | Partitioned By |
|---------|------------|---------------|
| id      | BIGINT     | No            |
| date    | DATE       | Yes           |
| region  | STRING     | No            |
| amount  | DECIMAL(10,2) | No          |

**Code:**

```sql
-- Example SQL for partitioning a table
CREATE TABLE sales (
    id BIGINT,
    date DATE,
    region STRING,
    amount DECIMAL(10,2)
) PARTITIONED BY (date);
```

**Example input / output:**

| Query            | Partition Pruning | Statistics Used | Cost Preview Result |
|------------------|-------------------|-----------------|--------------------|
| SELECT * FROM sales WHERE date >= '2023-01-01' AND date <= '2023-01-31' | Yes              | Yes             | $5,000.00          |

**Data:**

```json
{
  "partition_columns": ["date"],
  "statistics_interval": "daily",
  "budget_limits_per_team": {
    "team_a": 5000,
    "team_b": 7000
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User submits a query
6. Is the table partitioned?
7. Trino uses partition pruning
8. Are statistics available for thequery conditions?
9. Trino uses statistical estimatesto refine the plan
10. Trino runs a full scan, costing$X
11. Query optimizer provides costpreview and budget check
12. User receives real-time alerts ifcost exceeds limit

