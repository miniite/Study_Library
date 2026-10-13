---
type: "practice"
difficulty: "Deep"
---

# How would you design an online index rebuild to avoid blocking a 24x7 OLTP table, and what are the limitations of ONLINE = ON?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Clustered Indexes and Indexed Views](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an online index rebuild on a 24/7 OLTP (Online Transaction Processing) table in SQL Server while avoiding blocking, we need to understand the concept of "ONLINE" indexing and its limitations. The core idea is to leverage the ONLINE = ON option during the index rebuild process, which allows transactions to continue running without being blocked by the operation.

Firstly, let's clarify what ONLINE = ON means: it enables concurrent data modifications while the index is being rebuilt. This feature ensures that the table remains available for read and write operations during the index maintenance task. However, there are limitations and trade-offs associated with using this option, which we will explore.

Now, let's dive into designing an online index rebuild step by step:

Identify the Index: Determine which non-clustered index needs to be rebuilt.
Analyze the Impact: Assess how often the table is modified (inserts, updates, deletes) and the transactional load during peak hours.
Choose the Right Time: Schedule the rebuild at a time when the system has the least amount of activity. However, if this cannot be avoided due to business requirements, ONLINE = ON can still help manage concurrency.

Here is an example of how you might run an online index rebuild using T-SQL:

SQL
ALTER INDEX idx_example ON your_table
REBUILD WITH (ONLINE = ON);

Key considerations and limitations include:

Memory Usage: Online index rebuilding consumes more memory, which could impact other processes running on the server.
Locking and Blocking: While ONLINE = ON minimizes blocking, there can still be some level of lock contention. However, it is much better than traditional offline rebuilds where tables are locked entirely.
Performance Impact: The operation may have a slight performance impact due to the overhead involved in maintaining concurrency.

To mitigate these issues:

Ensure that sufficient resources (CPU, memory) are allocated during the rebuild process.
Monitor the server closely for any signs of resource contention and take corrective actions if necessary.

In conclusion, while ONLINE = ON provides a way to maintain index integrity without blocking transactions, it is not a panacea. Proper planning, monitoring, and resource management are crucial when performing such operations on a 24/7 OLTP table.

**Code:**

```sql
ALTER INDEX idx_example ON your_table
REBUILD WITH (ONLINE = ON);
```

