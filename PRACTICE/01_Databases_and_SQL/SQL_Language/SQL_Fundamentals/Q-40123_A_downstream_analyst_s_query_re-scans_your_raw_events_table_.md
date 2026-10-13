---
type: "practice"
difficulty: "Deep"
---

# A downstream analyst's query re-scans your raw events table 40 times a day. How would you decide between a materialized view, a scheduled aggregate table, and a warehouse result cache to cut cost?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a downstream analyst's query re-scanning your raw events table 40 times a day, we need to carefully consider the trade-offs between materialized views, scheduled aggregate tables, and warehouse result caches. Each option has its own advantages and drawbacks in terms of cost, performance, and maintenance. Let’s break down each approach step-by-step.

Firstly, a materialized view is essentially a precomputed result set that gets stored persistently in the database. It can be refreshed manually or automatically on schedule. This ensures that queries always return up-to-date results. However, materialized views require more storage space and might be costly to maintain if they are frequently updated.

Secondly, a scheduled aggregate table is created by running an expensive query at regular intervals to populate a separate table with summarized data. This reduces the number of times the raw events table needs to be queried but increases the load on your database during the scheduled refresh time. The cost savings come from reducing the number of reads against the raw events, potentially allowing you to use cheaper storage or compute resources.

Lastly, a warehouse result cache leverages caching mechanisms within the data warehouse itself. This can dramatically reduce read latency and improve query performance without storing any additional data persistently. However, it may not provide the exact same results as running queries against the raw events table every time due to its in-memory nature.

To decide between these options, we need to consider several factors: the frequency of updates, the cost of storage, the impact on database resources during refresh times, and the importance of query accuracy. Let’s reason through each approach from first principles:

Materialized View:

#### Pros
Always returns up-to-date results; reduces load on raw events table.

#### Cons
Requires more disk space for storing the materialized view; maintenance overhead.

Scheduled Aggregate Table:

#### Pros
Significant cost savings by reducing read operations against raw events.

#### Cons
May introduce latency if not perfectly aligned with query needs; requires scheduling and potential manual intervention.

Warehouse Result Cache:

#### Pros
Fastest results due to in-memory storage; minimal overhead during cache hits.

#### Cons
Might return stale data if the cache is not refreshed frequently enough; increased complexity due to caching logic.

Given that the analyst queries the raw events table 40 times a day, we should evaluate the following:

Frequency of Updates: If the raw events are updated infrequently (e.g., daily), a materialized view could be appropriate. However, if updates occur more frequently, an aggregate table or result cache might be better.

Storage Costs: Materialized views and scheduled aggregate tables consume additional storage space. The cost savings from reducing read operations with an aggregate table should be balanced against the storage costs.

Query Accuracy: A materialized view ensures that queries always return the latest data, while a cached result may not. If query accuracy is critical, a materialized view or regularly refreshed cache might be necessary.

#### Performance Considerations
The performance impact of refreshes (for aggregate tables and caches) should be evaluated. Frequent updates to raw events could require careful scheduling to minimize disruption.

In conclusion, the best approach depends on the specific context and requirements:

Use a materialized view if query accuracy is paramount and updates are infrequent.
Implement an aggregate table if read operations dominate costs and updates are less frequent.
Employ a result cache for high-performance, low-latency queries at the cost of potential staleness.

**Comparison:**

| Approach       | Pros                                                                         | Cons                                                                           |
|----------------|------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| Materialized View | Always returns up-to-date data; reduces raw event read load                  | Requires additional storage; maintenance overhead                             |
| Scheduled Aggregate Table  | Cost savings from reduced reads; less frequent update impacts                 | Potential for latency during refresh period                                    |
| Warehouse Result Cache    | High performance, minimal latency                                             | May return stale data if not frequently updated; increased complexity         |

**Code:**

```sql
-- Example SQL to create a materialized view (using PostgreSQL)
CREATE MATERIALIZED VIEW events_summary AS
SELECT * FROM raw_events;

-- Example SQL for scheduled aggregate table refresh (using PostgreSQL with cron schedule)
BEGIN TRANSACTION;
DELETE FROM aggregated_events;  -- Clear existing data
INSERT INTO aggregated_events SELECT ... FROM raw_events WHERE timestamp >= now() - interval '1 day';
COMMIT;
```

**Example input / output:**

```text
Input:
- Raw events table with 40 queries per day

Output (materialized view):
- Persistent stored summary of raw events
- Query always returns latest data without re-scanning

Output (scheduled aggregate table):
- Aggregated data refreshed daily, reducing read load on raw events
- Potential for latency during refresh period

Output (warehouse result cache):
- Fastest results due to in-memory caching
- May return stale data if not frequently updated
```

**Data:**

```json
{
    "raw_events": {
        "columns": ["event_id", "user_id", "timestamp", "action", "data"],
        "schema": "bigint, bigint, timestamp, text, jsonb"
    },
    "aggregated_events": {
        "columns": ["count_of_events", "total_actions", "max_timestamp"],
        "schema": "bigint, bigint, timestamp"
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Raw Events
6. Is Update Frequent?
7. Use Materialized View
8. Create Scheduled AggregateTable
9. Scheduled Refresh ->Aggregated Table
10. Do Queries Require LatestData?
11. Use Materialized View
12. Employ Warehouse ResultCache

