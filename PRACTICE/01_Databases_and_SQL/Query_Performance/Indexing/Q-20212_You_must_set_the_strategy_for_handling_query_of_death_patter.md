---
type: "practice"
difficulty: "Expert"
---

# You must set the strategy for handling 'query of death' patterns — rare parameter combinations that produce catastrophic plans and take down shared resources. Architect the org-wide defense combining plan-cost limits, statement timeouts, and index design.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with "query of death" (QoD) patterns—rare parameter combinations that can degrade system performance significantly—we need a robust strategy to ensure shared resources are not overwhelmed. Our approach will combine plan-cost limits, statement timeouts, and strategic index design to mitigate these issues while maintaining overall database efficiency.

#### Triaging the Question

The core of this question is about identifying and mitigating high-impact queries that could bring down the entire system. The interviewer is probing our understanding of query execution plans (QEP), resource management, cost-based optimization, and the interplay between different parameters such as CPU time, memory usage, and I/O operations.

#### Assumptions
Database Environment: PostgreSQL or a similar relational database with support for advanced query optimization.
Resource Limits: Defined limits on CPU, memory, and I/O resources per session or statement.
Monitoring Tools: Existing tools to track resource consumption and performance metrics.

#### First-Principles Solution
1. Plan-Cost Limits

We will implement plan-cost limits by setting thresholds for the estimated cost of query execution plans. This is achieved through the following steps:

Cost-Based Estimation: Enable cost-based optimization in the database.
Dynamic Thresholds: Set dynamic threshold values based on historical data and current system load.
Query Plan Analysis: Analyze each query plan to ensure it does not exceed the defined cost limits. If a plan exceeds this limit, we will either reject the query or suggest an optimized version.
SQL
-- Example SQL for setting cost-based optimization in PostgreSQL
ALTER DATABASE your_database_name SET enable_cost_based_optimizations = on;
2. Statement Timeouts

Statement timeouts are crucial to prevent long-running queries from monopolizing resources. We will implement:

Default Timeout Values: Set default statement timeout values.
Session-Level Configuration: Allow configuration of statement timeouts per session or user role based on historical data and risk assessment.
SQL
-- Example SQL for setting statement timeout in PostgreSQL
SET statement_timeout = 60000; -- 60 seconds
3. Index Design

Index design is critical to improving query performance and ensuring that high-impact queries do not cause resource bottlenecks. We will:

Identify High-Impact Queries: Use explain plans and performance metrics to identify queries causing significant load.
Create Specific Indexes: Develop index strategies tailored to these high-impact queries, such as covering indexes or composite indexes.
Regular Maintenance: Ensure regular maintenance of indexes to avoid degradation over time.
SQL
-- Example SQL for creating an index in PostgreSQL
CREATE INDEX idx_name ON table_name (column1, column2);
4. Monitoring and Alerting

Implement a monitoring system to track resource usage and performance metrics in real-time. This will help us proactively identify and mitigate potential issues.

Performance Metrics: Monitor CPU, memory, I/O, and query execution time.
Alerts: Set up alerts for critical thresholds that could lead to QoD.
5. Failover and Recovery

Ensure robust failover mechanisms are in place to handle resource exhaustion gracefully. This includes:

Automatic Scaling: Implement auto-scaling capabilities if possible.
Failover Strategies: Plan for failover scenarios where resources exceed predefined limits.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By combining plan-cost limits, statement timeouts, and strategic index design, we can effectively defend against "query of death" patterns. Key steps include enabling cost-based optimization, setting appropriate timeout values, creating specific indexes, monitoring resource usage, and implementing failover strategies.

#### Common mistakes
to avoid include:

Over-reliance on query optimization without proper monitoring.
Ignoring the dynamic nature of system load and failing to adjust thresholds accordingly.
Failing to implement comprehensive failover mechanisms that can handle resource exhaustion.

#### Follow-ups
might involve discussing specific tools for cost-based estimation, more advanced index design techniques like materialized views, or integration with cloud services for auto-scaling capabilities.

**Comparison:**

| Query | Cost Threshold (in $) |
|-------|----------------------|
| Q1    | 2                    |
| Q2    | 2.5                  |
| Q3    | 4                    |

**Code:**

```sql
-- Example SQL configurations and commands
ALTER DATABASE your_database_name SET enable_cost_based_optimizations = on;
SET statement_timeout = 60000; -- 60 seconds

CREATE INDEX idx_name ON table_name (column1, column2);
```

**Example input / output:**

```text
| Query | Plan Cost (in $) |
|-------|-----------------|
| Q1    | 0.5             |
| Q2    | 1.2             |
| Q3    | 3.0             |

For Q3, the plan cost exceeds the threshold, so we will optimize or reject it.
```

**Data:**

```json
{
  "database": {
    "name": "your_database",
    "settings": {
      "enable_cost_based_optimizations": true,
      "statement_timeout": 60000,
      "indexing_strategy": ["covering", "composite"]
    }
  },
  "queries": [
    {"query_plan": "EXPLAIN SELECT * FROM table_name WHERE column1 = 1 AND column2 = 2;"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Execution
4. Plan Cost > Threshold?
5. Reject or Optimize Query
6. Continue with Query
7. Monitor and Alert on ResourceUsage
8. Process Query Normally

