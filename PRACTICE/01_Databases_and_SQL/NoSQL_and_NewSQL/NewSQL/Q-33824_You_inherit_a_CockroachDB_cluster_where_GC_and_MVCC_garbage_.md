---
type: "practice"
difficulty: "Expert"
---

# You inherit a CockroachDB cluster where GC and MVCC garbage are ballooning storage; explain how gc.ttlseconds, protected timestamps, and long-running changefeeds interact, and how you'd remediate.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we have a CockroachDB cluster where garbage collection (GC) and Multi-Version Concurrency Control (MVCC) are causing excessive storage usage. To effectively address this issue, we need to understand how gc.ttlseconds, protected timestamps, and long-running changefeeds interact with each other. We'll then devise a remediation strategy based on these interactions.

Firstly, let's briefly summarize the key concepts:

gc.ttlseconds: Determines how long data is retained in CockroachDB before it is eligible for garbage collection.
Protected Timestamps: Ensures that certain transactions are not garbage collected even if their TTL has expired.
Long-Running Changelogs (Changefeeds): Allows real-time access to changes in the database, which can contribute to increased storage usage.

Now, let's delve into how these components interact and what remediation strategies we might employ.

Interactions
gc.ttlseconds

gc.ttlseconds is a critical parameter that controls the garbage collection process. Data older than this timestamp will be eligible for cleanup. If this value is set too low or if it's not managed properly, unnecessary data retention can lead to increased storage consumption. For example, if there are many long-running transactions (which might hold onto data longer due to protected timestamps), setting gc.ttlseconds too aggressively could result in excessive data removal.

Protected Timestamps

Protected timestamps prevent certain transaction records from being garbage collected even if they have exceeded their gc.ttlseconds. This is useful for maintaining the integrity of historical transactions, but it can also lead to bloated storage. If these timestamps are not managed properly—such as by cleaning up old protected transactions—they can contribute significantly to storage issues.

Long-Running Changelogs

Changefeeds create a persistent record of all changes in the database. While this feature is invaluable for real-time data access, it also consumes significant storage over time. Long-running changefeeds that capture every single change can quickly lead to excessive storage usage if not managed carefully.

Remediation Strategy

To address these issues, we need to take a multi-faceted approach:

Adjust gc.ttlseconds: Ensure this value is set appropriately based on the application's requirements and data retention policies. A common practice is to set it to a few weeks or months, depending on how long historical data needs to be retained.

Manage Protected Timestamps: Regularly clean up old protected timestamps that are no longer needed. This can be done by running INACTIVE transactions that mark old protected timestamps as inactive and allow them to be garbage collected.

Optimize Changefeeds:

Use WITH TRUNCATE when creating changefeed sources to avoid unnecessary duplication of data.
Monitor the size of changefeeds and consider pruning older entries if they are not being accessed frequently.
Implement quotas or limits on the number of changefeeds per node to prevent over-consumption of storage.

Implement Data Archiving: For very old, infrequently accessed data, consider archiving it to a separate, less costly storage solution. CockroachDB supports multi-cluster replication and can be used for this purpose.

Monitor and Tune Regularly: Set up regular monitoring and tuning schedules to ensure that the cluster is operating efficiently. Use CockroachDB’s built-in tools like pg_dump and pg_restore to manage large-scale data migrations and cleanups.

#### Closing Summary

By understanding the interactions between gc.ttlseconds, protected timestamps, and long-running changefeeds, we can effectively remediate storage issues in a CockroachDB cluster. Properly setting parameters, managing protected transactions, optimizing changefeeds, implementing data archiving, and regular monitoring are key steps to maintaining efficient storage usage.

#### Common Mistakes
Setting gc.ttlseconds too low or ignoring it entirely.
Failing to regularly clean up old protected timestamps.
Not optimizing changefeeds for size and frequency of use.
Overlooking the importance of data archiving, especially for older, infrequently accessed data.

**Code:**

```

-- Example SQL command to adjust gc.ttlseconds ALTER SYSTEM SET 'gc.ttlseconds' TO 2592000; -- Adjust this value based on your retention policy (e.g., 30 days)

CODE
Copy

```sql
-- Example SQL command to mark old protected timestamps as inactive
UPDATE pg_catalog.inactive_transactions SET active = false WHERE timestamp < NOW() - INTERVAL '1 year';
```

**Flow:**

1. Identify storage issues
2. Adjust gc.ttlseconds
3. Monitor and tune regularly
4. Manage protected timestamps
5. Optimize changefeeds
6. Implement data archiving
7. Maintain efficient clusteroperation

