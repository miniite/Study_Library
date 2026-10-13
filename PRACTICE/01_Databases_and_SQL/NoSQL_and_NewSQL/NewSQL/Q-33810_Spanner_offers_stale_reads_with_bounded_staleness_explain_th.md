---
type: "practice"
difficulty: "Deep"
---

# Spanner offers stale reads with bounded staleness; explain the consistency semantics, how staleness bounds interact with TrueTime, and when you'd use exact-staleness reads for a read-heavy analytics path.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Spanner, stale reads provide query results that are guaranteed to be no older than a specified amount of time in the past. The "bounded staleness" feature allows you to specify this bound using the TRUE_TIMESTAMP() function or by setting a bounded staleness value directly through a read option. This capability is particularly useful for applications requiring strong consistency guarantees while still allowing some level of data freshness trade-off.

Spanner uses TrueTime, an API that provides the current time and the latest committed timestamp in the database history, to ensure accurate bounds on staleness. When you perform a stale read with bounded staleness, Spanner ensures that the results are no older than the specified bound according to TrueTime.

TrueTime interacts with bounded staleness by providing a precise time reference for your queries. For example, if you set a staleness bound of 5 seconds, and the latest committed timestamp is 10 seconds ago, TrueTime will enforce that the read result cannot be more than 5 seconds old relative to this current timestamp.

Exact-staleness reads are particularly useful in scenarios where you need highly consistent data for analytics or reporting purposes. Since Spanner guarantees that exact-staleness reads reflect a specific historical point in time, they can provide accurate and consistent results, making them ideal for read-heavy analytics paths.

To illustrate when to use exact-staleness reads:

Financial Reporting: For generating financial reports, you might want to ensure that all transactions considered are those committed up to 5 seconds before the report is generated. This guarantees consistency in reported figures.
Real-time Analytics Pipelines: In pipelines where real-time data updates are critical but historical accuracy is also necessary for certain checks or reporting periods, exact-staleness reads can provide a way to balance these requirements.

Let's break down how you might use this feature in practice:

Setting the Staleness Bound:

SQL
SELECT * FROM table WHERE timestamp <= @timestamp_bound;

Using TRUE_TIMESTAMP() for Exact Staleness:

SQL
SELECT * FROM table WHERE timestamp <= TRUE_TIMESTAMP() - INTERVAL '5' SECONDS;

Scheduling Queries with TrueTime Boundaries:

SQL
CREATE ATOMIC TRANSACTION WITH TIMESTAMP BOUND @timestamp_bound;

The key considerations when using exact-staleness reads are:

Data Freshness: While you get strong consistency, the data might be older than real-time for applications requiring very recent updates.
Resource Utilization: Exact-staleness queries can consume more resources due to the additional overhead of verifying the timestamp bounds and maintaining historical accuracy.

**Code:**

```sql
-- Example query with exact staleness using TRUE_TIMESTAMP()
SELECT * FROM sales WHERE order_time <= TRUE_TIMESTAMP() - INTERVAL '5' SECONDS;
```

