---
type: "practice"
difficulty: "Deep"
---

# What is a partial index and give a use case for it.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A partial index in PostgreSQL allows you to create an index that only includes a subset of rows from a table, based on a specific condition. This can significantly reduce the size of the index and improve query performance by filtering out unnecessary data upfront. For example, if you have a users table with millions of entries but mostly read queries are about active users (e.g., those who logged in within the last 30 days), you could create a partial index on the last_login_date column for only those rows where last_login_date > now() - INTERVAL '30 days'.

To triage this question, the interviewer is probing your understanding of indexing principles and how to optimize query performance. They are also likely interested in your ability to identify appropriate use cases and apply them effectively.

Assuming you have a users table with columns such as id, username, email, last_login_date, etc., you might want to consider creating a partial index on the last_login_date. Here’s how you can create it:

SQL
CREATE INDEX idx_users_recent_activity ON users (last_login_date)
WHERE last_login_date > now() - INTERVAL '30 days';

Let's walk through this step by step from first principles.

Understanding Indexes: First, recall that indexes in databases help speed up query performance by allowing the database to quickly locate rows based on certain columns. However, they come with a trade-off: creating and maintaining an index consumes additional storage space and can slow down write operations (INSERTs, UPDATEs, DELETEs).

Partial Indexes: The key idea behind partial indexes is that you only create an index for a subset of the data. In this case, we are indexing only those rows where last_login_date falls within the last 30 days.

Use Case Analysis:

Identifying Active Users: Often, applications need to query active users frequently (e.g., for notifications or analytics). By creating a partial index on these active records, you can significantly reduce the number of rows the database needs to scan.
Reducing Index Size: If your table has many inactive users who rarely interact with the system, creating an index that includes only recent activity can greatly reduce its size and thus improve read performance.

Creating the Partial Index:

You would first identify which columns you want to include in the index.
Then, use a WHERE clause to specify the condition for including rows in the index.
Finally, execute the SQL statement as shown above.

Benefits and Considerations:

Performance Improvement: Queries that filter on the last_login_date will benefit from this index, leading to faster execution times.
Maintenance Cost: While creating indexes can improve performance, they also require maintenance. Ensure that the cost of maintaining the partial index is justified by the performance benefits.

#### Common Mistakes

Failing to identify appropriate conditions for the WHERE clause can lead to an index that does not provide significant performance improvements.
Over-indexing can degrade write performance and increase storage costs unnecessarily.

**Code:**

```sql
CREATE INDEX idx_users_recent_activity ON users (last_login_date)
WHERE last_login_date > now() - INTERVAL '30 days';
```

