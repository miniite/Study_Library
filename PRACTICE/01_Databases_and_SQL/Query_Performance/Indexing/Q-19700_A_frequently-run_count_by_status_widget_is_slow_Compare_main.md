---
type: "practice"
difficulty: "Deep"
---

# A frequently-run 'count by status' widget is slow. Compare maintaining a triggered counter table, a partial index per status, and an approximate count. Which fits a real-time UI?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a slow "count by status" query in a frequently-run widget, we need to compare three potential solutions: maintaining a triggered counter table, creating partial indexes per status, and using an approximate count. Each approach has its trade-offs in terms of complexity, performance, and accuracy. Let's break down each solution from first principles.

First, let’s summarize the requirements and constraints:

Real-time UI: The query needs to provide up-to-date counts quickly.
Frequent Runs: The query runs often enough that performance is a critical concern.
Status Tracking: We are counting records by their status, which could be an enum or string field.

#### Triage

The question is essentially asking us to evaluate different strategies for efficiently tracking and querying the count of records based on their status. The interviewer likely wants to understand your ability to analyze performance trade-offs, especially in a real-time context.

Maintaining a Triggered Counter Table

Maintaining a separate counter table involves incrementing or decrementing counters whenever an insert, update, or delete operation changes the record's status. This ensures that the count is always accurate and up-to-date.

#### Implementation
Each time a record's status changes, you would atomically update the corresponding entry in the counter table.
Advantages:
Accuracy: The count will be exact at all times.
Simplicity for Reporting: Direct queries on the counter table are straightforward and fast.
Disadvantages:
Overhead: Every update to a status involves an additional write operation, which can impact performance if updates occur frequently.
Complexity in Synchronization: Ensuring that all statuses are updated correctly is crucial.
Creating Partial Indexes Per Status

Partial indexes allow you to create indexes on specific columns with certain values. For this scenario, you could create a partial index for each status value:

#### Implementation
SQL
CREATE INDEX idx_status_0 ON table_name (id) WHERE status = 0;
CREATE INDEX idx_status_1 ON table_name (id) WHERE status = 1;
Advantages:
Performance: Queries can use the appropriate index based on the status, potentially reducing read times.
Disadvantages:
Storage Overhead: Each partial index requires additional storage space.
Complexity in Maintenance: Managing and keeping the indexes up-to-date as new statuses are added or existing ones change could be cumbersome.
Using an Approximate Count

Approximate counts can provide near-real-time updates without the overhead of maintaining a counter table. This approach uses techniques like sampling, reservoir sampling, or sketching algorithms.

#### Implementation
Use statistical methods to estimate the count.
Advantages:
Low Overhead: Minimal impact on write operations since no additional counters are maintained.
Flexibility: Can be implemented with minimal changes to your existing database schema.
Disadvantages:
Accuracy: The counts will not be exact and may vary over time, leading to potential discrepancies.

#### Conclusion

For a real-time UI that requires frequent updates, maintaining a triggered counter table is likely the best choice if performance overhead can be tolerated. It ensures accuracy and simplicity for reporting. However, if the write operations are extremely frequent or if storage costs are a concern, partial indexes per status could be a good compromise, offering better read performance without the high cost of updates.

If real-time accuracy is not critical and you prioritize low write overhead, approximate counts can provide a viable solution but with potential inaccuracies in reporting.

#### Common Mistakes
and Follow-Ups
Overlooking Write Overhead: Forcing an update on every status change might not be necessary; consider caching mechanisms.
Ignoring Index Maintenance Costs: Partial indexes require careful management to avoid performance degradation.
Failing to Consider Approximation Errors: In cases where exact counts are essential, approximate methods may not be suitable.

**Code:**

```sql
-- Example of partial index creation
CREATE INDEX idx_status_0 ON table_name (id) WHERE status = 0;
CREATE INDEX idx_status_1 ON table_name (id) WHERE status = 1;
```

