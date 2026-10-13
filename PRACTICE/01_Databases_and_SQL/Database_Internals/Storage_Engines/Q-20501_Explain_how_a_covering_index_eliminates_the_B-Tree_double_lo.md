---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a covering index eliminates the B-Tree double lookup (secondary index then clustered index). When would you deliberately not create a covering index despite the read win?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a covering index eliminates the B-Tree double lookup, we need to first comprehend what a B-Tree is and how it works in the context of databases. A B-Tree is a balanced tree data structure that maintains sorted data and allows for efficient search, insertion, and deletion operations. In database systems, B-Trees are often used as the index structure for clustered indexes.

A covering index is an index where all the columns required by a query are stored within the index itself, meaning no additional lookups to the actual table rows are needed. This eliminates the need for a second lookup in the B-Tree structure (which would otherwise be necessary if the required data is not contained within the index).

Here's how it works: Consider a scenario where you have a users table with columns such as id, name, email, and created_at. If your query needs both name and email, using an index that covers these two fields will avoid a second lookup to retrieve the row from the actual users table. The B-Tree double lookup would involve first finding the id of the user in the secondary index, then looking up the corresponding row in the users table.

The key idea here is that by including all necessary columns in the covering index, you reduce the number of I/O operations required to satisfy a query. This can significantly improve performance and scalability in read-heavy workloads.

Now, let's consider when not to create a covering index despite these benefits. One common scenario where this might be the case is during write-heavy operations. Adding columns to an index increases the size of the index itself, which means every write operation (insert, update, delete) will require more storage and potentially slower write times as the B-Tree structure needs to be updated accordingly.

Another reason could be space constraints. If your database has limited storage, adding additional index columns might not be feasible due to the overhead they introduce. For example, a large dataset with many non-indexed columns would benefit from covering indexes, but if storage is at a premium, you may need to weigh this against other optimizations.

Let's take an example. Suppose we have a transactions table where each row represents a financial transaction. The table has the following columns: id, user_id, amount, and timestamp. A common query might be to retrieve all transactions for a specific user within a date range, requiring both user_id and amount.

A covering index on (user_id, amount) would include these two fields in the B-Tree structure. When querying for transactions by user_id and amount, this index allows us to directly find all necessary information without needing to go back to the actual table rows.

However, if we are frequently inserting or updating new transaction data, creating a covering index on (user_id, amount) could slow down these operations due to increased I/O overhead. In such cases, it might be more appropriate to have separate indexes on user_id and amount, allowing for faster writes at the cost of slightly slower reads.

In summary, while covering indexes can significantly enhance read performance by reducing B-Tree double lookups, there are scenarios where maintaining a smaller set of more specialized indexes or even no index might be the better choice.

