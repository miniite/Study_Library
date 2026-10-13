---
type: "practice"
difficulty: "Deep"
---

# How would you measure the abort/retry rate under SERIALIZABLE in production and set an SLO around it? What signals tell you the level is too aggressive?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the abort/retry rate under SERIALIZABLE isolation level in a production database, we need to understand that this isolation level ensures serializability but can lead to more aborts and retries due to its strict nature. The primary goal is to set Service Level Objectives (SLO) on these rates to ensure they remain within acceptable bounds without compromising the system's availability or performance.

#### Triaging the Question

The question probes your understanding of SERIALIZABLE isolation, how it impacts transactional behavior, and the ability to monitor and control abort/retry operations. The key assumptions include:

You have access to relevant monitoring tools.
The database environment is already in a production state with SERIALIZABLE transactions enabled.
There are no known bugs or performance issues that could skew results.

#### First Principles Solution

To measure the abort/retry rate under SERIALIZABLE isolation, we start by identifying the transaction states and the conditions under which retries occur. Then, we establish metrics and thresholds to ensure these rates remain within acceptable limits.

Identify Transaction States:

Commit: The transaction successfully completes.
Abort (Rollback): The transaction is rolled back due to a conflict or other issue.
Retry: A transaction that fails attempts to retry up to a certain number of times before ultimately aborting.

Monitoring Tools and Metrics:

Use database management system (DBMS) metrics tools like Prometheus, Grafana, or custom dashboards.
Monitor the following key performance indicators (KPIs):
Number of transactions per second (TPS).
Abort rate: Percentage of transactions that roll back.
Retry rate: Percentage of transactions that retry before committing.

Setting SLO:

Define acceptable abort and retry rates based on historical data, industry standards, or business requirements.
For example, if the current system operates at an abort rate of 0.5% and a retry rate of 1%, these could be your baseline metrics.
Set thresholds such as: Abort Rate < 2%; Retry Rate < 3%.

Detection of Aggressive Behavior:

Monitor for sudden spikes in abort or retry rates, which might indicate issues like performance degradation or concurrency problems.
Use anomaly detection algorithms to flag unusual patterns that could signal a problem.

Response Strategy:

If the rates exceed the SLO, investigate potential causes such as schema changes, index updates, or increased load.
Implement optimizations like improving query performance, adding indexes, or fine-tuning isolation levels.

#### Closing Summary
and Common Mistakes

The key to managing abort/retry rates under SERIALIZABLE is a robust monitoring system that can quickly identify issues. Setting realistic SLOs based on historical data helps prevent unnecessary changes and ensures the database remains performant. Common mistakes include:

Ignoring the context of transaction types (read-heavy vs write-heavy).
Not having adequate logging or monitoring in place.
Setting overly strict thresholds that could lead to frequent false alarms.
Triangulation with Other Concepts

Understanding how SERIALIZABLE interacts with other concepts like database indexing, query optimization, and workload management is crucial. For instance, aggressive retry rates might indicate poor query performance; optimizing queries can reduce retries while maintaining serializability.

**Comparison:**

```
Metric	Value
Abort Rate	0.5%
Retry Rate	1.0%
Total TPS	100000
```

**Code:**

```sql
-- Example SQL to count aborts and retries over a period
SELECT
    COUNT(*) as total_transactions,
    SUM(CASE WHEN status = 'ABORTED' THEN 1 ELSE 0 END) as aborted_transactions,
    SUM(CASE WHEN status = 'RETRY' THEN 1 ELSE 0 END) as retried_transactions
FROM transactions
WHERE timestamp >= '2023-10-01'
AND timestamp < '2023-10-31';
```

**Example input / output:**

```text
Input:
SELECT
    COUNT(*) as total_transactions,
    SUM(CASE WHEN status = 'ABORTED' THEN 1 ELSE 0 END) as aborted_transactions,
    SUM(CASE WHEN status = 'RETRY' THEN 1 ELSE 0 END) as retried_transactions
FROM transactions
WHERE timestamp >= '2023-10-01'
AND timestamp < '2023-10-31';

Output:
{
  "metrics": {
    "abort_rate": 0.5,
    "retry_rate": 1.0,
    "total_transactions": 100000,
    "timestamp_range": "2023-10-01 to 2023-10-31"
  }
}
```

**Data:**

```json
{
  "metrics": {
    "abort_rate": 0.5,
    "retry_rate": 1.0,
    "total_transactions": 100000,
    "timestamp_range": "2023-10-01 to 2023-10-31"
  }
}
```

**Flow:**

1. Transaction
2. Abort?
3. Abort
4. Retry
5. Alert
6. Retry Count++
7. Max Retries Exceeded?
8. Abort
9. Retry

