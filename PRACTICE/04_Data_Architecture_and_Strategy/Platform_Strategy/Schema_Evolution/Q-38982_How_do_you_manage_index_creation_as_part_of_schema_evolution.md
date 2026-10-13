---
type: "practice"
difficulty: "Deep"
---

# How do you manage index creation as part of schema evolution on hot tables — concurrent index builds, invalid-index cleanup, write amplification, and the decision to build during off-peak vs anytime?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage index creation as part of schema evolution on hot tables, you need a comprehensive strategy that addresses concurrent index builds, invalid-index cleanup, write amplification, and timing decisions based on system load. At an Architect/Principal level, this involves understanding the underlying database architecture, performance implications, and business requirements.

#### Triaging the Question

The question is asking for a detailed approach to handle indexes during schema evolution on high-write tables. Specifically:

Concurrent Index Builds: How do you manage index creation without causing blocking or performance degradation?
Invalid-Index Cleanup: What steps are taken when an index becomes invalid, and how do you ensure data integrity?
Write Amplification: How do you minimize the additional write operations caused by index updates?
Timing Decisions: When should indexes be built—during off-peak hours or continuously?

#### Assumptions
The database is a modern relational system (e.g., PostgreSQL, MySQL).
Indexes are used for performance optimization and query speed.
Schema evolution involves adding, modifying, or dropping indexes.

#### First Principles Solution
1. Concurrent Index Builds: Using Asynchronous Index Creation

#### Approach
Use background tasks to create indexes asynchronously. This can be achieved through job schedulers like Apache Airflow or database-specific features such as index creation via CREATE INDEX CONCURRENTLY in PostgreSQL.

#### Implementation
Example:
SQL
-- For PostgreSQL, use CREATE INDEX CONCURRENTLY
CREATE INDEX CONCURRENTLY idx_name ON table_name(column_name);
Benefits: Minimizes the impact on transactional workloads by allowing indexes to be built in the background.
2. Invalid-Index Cleanup: Monitoring and Automated Rebuilding

#### Approach
Implement a monitoring system that tracks index health, such as using pg_stat_user_indexes or similar metrics.
Steps:
Periodically check for invalid indexes.
Rebuild them when necessary to ensure data integrity.
3. Write Amplification: Minimizing Index Updates

#### Approach
Reduce the number of write operations by carefully designing index strategies and batch processing.
Techniques:
Use partial indexes where appropriate.
Batch updates to minimize locking and contention.
4. Timing Decisions: Off-Peak vs Continuous Builds
Off-Peak Builds: Schedule index creation during periods of low transactional activity, such as late at night or weekends.
Continuous Builds: Monitor system load and schedule index builds when the impact is minimal based on real-time metrics.

#### Closing Summary

In summary, managing indexes during schema evolution requires a multi-faceted approach. By leveraging asynchronous index creation, automated monitoring for invalid indices, minimizing write amplification, and timing builds strategically, you can ensure that index management does not degrade transactional performance. The key is to balance the trade-offs between performance, data integrity, and operational feasibility.

#### Common Mistakes/Follow-Ups
Mistake: Failing to monitor index health and rebuilding them when necessary.

#### Follow-Up
How would you design a system to automatically detect invalid indexes and trigger rebuilds?
Mistake: Not considering the impact of write amplification on overall performance.

#### Follow-Up
Can you provide an example of how partial indexes can be used effectively in your database schema?

**Comparison:**

| Index Name | Column(s) | Type       | Status      |
|------------|-----------|------------|-------------|
| idx_name1  | col_a, col_b | UNIQUE     | Valid       |
| idx_name2  | col_c     | B-tree     | Invalid     |
| idx_name3  | col_d     | GIN        | Valid       |

**Code:**

```sql
-- Example SQL for creating a concurrent index
CREATE INDEX CONCURRENTLY idx_name ON table_name(column_name);
```

**Example input / output:**

```text
Input: A schema evolution task that requires creating a new index on a hot table.
Output:
Step 1: Schedule the index creation using a background job at an off-peak hour (e.g., 2 AM).
Step 2: Monitor the system load and performance metrics to ensure minimal impact during the build.
Step 3: Validate the newly created index for correctness and data integrity.
```

**Data:**

```json
{
  "index_metrics": {
    "valid_indices": ["idx_name1", "idx_name2"],
    "invalid_indices": ["idx_name3"]
  },
  "system_load": {
    "hourly_transactions": [500, 600, 700],
    "off_peak_hours": ["00:00-04:00", "23:00-01:00"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Index Valid?
5. Index Valid
6. Schedule Index Rebuild
7. Schedule Index CreationDuring Off-Peak Hours
8. Migrate Invalid Index to NewTable and Drop Old

