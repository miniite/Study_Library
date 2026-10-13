---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a materialized view and a database snapshot historically in Oracle, and does the terminology matter today?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized View Refresh and Maintenance](<../../../../01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary:

Materialized views (MV) and database snapshots were both used to store precomputed results of queries at different points in time, but they serve slightly different purposes and have distinct behaviors. Historically, Oracle supported both concepts, with database snapshots being a more recent addition. Today, the terminology might not be as relevant due to Oracle's evolution towards Materialized Views, which offer similar functionality.

#### Triaging the Question

The question is asking for an explanation of two historical database concepts—materialized views and database snapshots—and whether their distinctions are still meaningful in modern Oracle databases.

Key Assumptions:
Historical Context: The interviewer expects knowledge about how these features were implemented historically.
Current State: There's a need to understand if these terms have any significance today, especially with the evolution of Oracle’s database management system (DBMS).
Step-by-Step First-Principles Solution:
Materialized Views in Oracle

Materialized views are precomputed query results stored as tables or materialized indexes. They are essentially cached versions of a complex query result that can be updated periodically, ensuring that the data reflects a consistent state at the time of the last update.

Purpose:

To reduce the cost of executing expensive queries by storing their results.
To provide fast access to frequently used or summarized data without hitting the database repeatedly.

Operation:

Materialized views can be refreshed manually, periodically (e.g., at specific times), or on demand.
They are optimized for read-heavy workloads and are particularly useful in scenarios where query performance is critical.

Types of Refreshes:

Complete refresh: Recomputes the entire view from scratch.
Incremental refresh: Updates only the rows that have changed since the last update, using a timestamp or a WHERE clause to identify changes.
Database Snapshots in Oracle

Database snapshots were introduced later as part of Oracle’s Data Guard feature. They are read-only copies of the database at specific points in time, which can be used for reporting and analysis without impacting the primary database.

Purpose:

To provide a consistent snapshot of the database state at various times.
To enable high availability and disaster recovery scenarios where multiple instances need to access historical data.

Operation:

Snapshots are created through logical or physical copies, depending on the configuration.
They can be read by multiple sessions simultaneously without locking the underlying primary database.
Snapshots are refreshed automatically as changes occur in the primary database and are not recomputed like materialized views.

Use Cases:

For reporting where real-time data is less critical, but historical data needs to be available.
To support point-in-time recovery without manual backup procedures.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

While both materialized views and database snapshots aim to provide efficient access to precomputed data, they serve different use cases. Materialized views are ideal for optimizing complex queries, whereas database snapshots excel in providing consistent historical data in a read-only manner without impacting the primary database.

In modern Oracle databases, the term "database snapshot" is less frequently used due to the introduction of more comprehensive features like GoldenGate and Data Guard, which offer similar functionality. However, understanding both concepts remains valuable for comprehending the full breadth of Oracle's data management capabilities.

#### Common Mistakes
Confusion between Refresh Mechanisms: Forgetting that materialized views can be refreshed incrementally or completely, while database snapshots are automatically refreshed.
Mixing Use Cases: Using a materialized view where a snapshot would suffice (or vice versa) due to misunderstanding the purpose of each.
Triaging Follow-Ups:
The interviewer might ask about specific scenarios where one type of feature would be preferred over the other.
They could inquire about performance implications and potential use cases for each.

**Comparison:**

|Product ID | Total Sales|
|----------|-----------|
|101       |   5000     |
|102       |   7000     |

**Code:**

```sql
-- Example of creating a materialized view in Oracle
CREATE MATERIALIZED VIEW sales_summary
REFRESH COMPLETE ON DEMAND AS
SELECT product_id, SUM(sales_amount) as total_sales
FROM sales_history
GROUP BY product_id;

-- Example of creating a database snapshot (not exact syntax)
BEGIN
  DBMS_SNAPSHOT.CREATE_SNAPSHOT( 'MY_SNAPSHOT' );
END;
```

**Example input / output:**

```text
Input:
- Sales data for product_id 101 and 102.

Output:
- Materialized view `sales_summary` with total sales of $5000 for product_id 101, and $7000 for product_id 102.
```

**Data:**

```json
{
  "sales_summary": {
    "product_id": [101, 102],
    "total_sales": [5000, 7000]
  }
}
```

**Flow:**

1. No
2. Yes
3. User Query
4. Is Data Fresh?
5. Retrieve from MaterializedView
6. Refresh Materialized View
7. Retrieve from MaterializedView
8. Report

