---
type: "practice"
difficulty: "Intermediate"
---

# How do you refresh a materialized view and what is the difference between complete and incremental refresh?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this discussion, we'll explore how to refresh a materialized view in a database system and delve into the differences between complete and incremental refresh methods. Materialized views are precomputed summaries of data that can be refreshed manually or automatically based on their refresh type.

#### Triage
The question revolves around understanding the process of refreshing materialized views, specifically distinguishing between complete and incremental refreshes. We'll assume that you have a basic understanding of what materialized views are and how they function within a database.

Step-by-Step Solution:

Understanding Materialized Views:

A materialized view is a precomputed summary or snapshot of data, stored as a physical table in the database.
It allows for faster query execution by storing the results of expensive queries, reducing the need to repeatedly run these queries.

Refreshing Materialized Views:

When data changes in the underlying tables that the materialized view depends on, it becomes necessary to refresh the materialized view to reflect the new data accurately.
There are two primary methods for refreshing a materialized view: complete and incremental.

Complete Refresh:

A complete refresh rebuilds the entire materialized view from scratch using the latest data from the underlying tables.
This method ensures that all data in the materialized view is up to date but can be time-consuming, especially for large datasets.

Incremental Refresh:

An incremental refresh updates only the parts of the materialized view that have changed since the last refresh.
It is more efficient than a complete refresh because it minimizes the computational overhead and reduces downtime during the refresh process.

#### Implementation
Differences:

Complete Refresh: Typically involves dropping the materialized view, running an SQL query to recompute all rows, and then recreating the view.
SQL
DROP MATERIALIZED VIEW mv_example;
CREATE MATERIALIZED VIEW mv_example AS SELECT ... FROM ...
Incremental Refresh: Usually involves a simpler update statement that applies changes from the source tables directly to the materialized view.
SQL
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_example;

#### Trade-offs

Complete refreshes are straightforward and ensure up-to-date data but can be resource-intensive, especially in high-traffic environments.
Incremental refreshes are more efficient and cause less disruption to the system but require careful management of the change detection mechanism.

#### Example Scenario
s:

If you have a large materialized view with millions of rows, performing a complete refresh might take several minutes or even hours, affecting overall database performance.
For a smaller view that changes frequently but only in small increments, an incremental refresh would be more suitable and maintain better performance.

Best Practices:

Choose the appropriate refresh type based on the size of the materialized view, the frequency of data updates, and the system’s load.
Implement monitoring to track when refreshes are necessary and ensure that the chosen method does not cause unnecessary disruptions.

#### Common Mistakes

Forgetting to update all dependent views or indexes during a complete refresh can lead to inconsistencies.
Incorrectly setting up an incremental refresh mechanism, such as missing triggers or incorrect synchronization logic, can result in outdated data being stored in the materialized view.

#### Closing Summary
Understanding and managing the process of refreshing materialized views is crucial for maintaining accurate and performant database queries. The choice between complete and incremental refreshes depends on factors like the size of the data, update frequency, and system performance requirements.

**Code:**

```sql
-- Example of a Complete Refresh
DROP MATERIALIZED VIEW mv_example;
CREATE MATERIALIZED VIEW mv_example AS SELECT ... FROM ...

-- Example of an Incremental Refresh
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_example;
```

