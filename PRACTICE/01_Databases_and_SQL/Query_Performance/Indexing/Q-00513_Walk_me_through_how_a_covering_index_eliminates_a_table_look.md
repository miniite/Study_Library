---
type: "practice"
difficulty: "Deep"
---

# Walk me through how a covering index eliminates a table lookup and when you'd choose one over a plain secondary index.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Covering Indexes and Index Strategy for Sharding](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/Covering_Indexes_and_Index_Strategy_for_Sharding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a covering index eliminates a table lookup, we first need to grasp the concept of indexing in databases. A covering index is a type of secondary index that includes all the columns needed by a query, thereby avoiding an additional lookup into the base table. This can significantly improve performance for read-heavy operations where the database needs to fetch multiple columns from the same table.

A plain secondary index only contains keys and pointers to the actual data in the table rows, so while it’s faster than a full table scan, it still requires an extra step to retrieve the complete row data. A covering index, on the other hand, can provide all necessary data directly from the index itself, reducing or even eliminating the need for additional reads.

Let's break down how this works and when you might choose a covering index over a plain secondary index:

#### Triaging the Question
What’s being asked: The core of the question revolves around understanding the mechanics of indexing in databases and the specific benefits of a covering index.
What the interviewer is probing: They want to ensure you understand the trade-offs between different types of indexes, particularly focusing on performance implications for read operations.
Step-by-Step Solution from First Principles
Understanding Indexes:
A secondary index is an additional structure that stores keys and pointers to actual data in a table.
The database uses these indexes to quickly locate rows based on the indexed column(s).
Covering Indexes:
A covering index includes all columns used by the query, thus providing all necessary information directly from the index itself.
Eliminating Table Lookups:
When a query can be satisfied entirely from an index (covering), there is no need to access the actual table rows, which significantly reduces I/O operations and speeds up execution.

#### Example Scenario
s

Let’s consider a scenario where you have a Customer table with several columns: customer_id, name, email, address, phone_number. A query might be SELECT name, email FROM Customer WHERE customer_id = 123.

Plain Secondary Index:
If we only create an index on customer_id, the database can quickly find the row pointer to Customer with id = 123.
However, it still needs to read from the actual table to get name and email.
Covering Index:
By creating a covering index that includes customer_id, name, and email, the database can fetch all required data directly from the index without touching the table.
Key Considerations
Index Maintenance Cost: Covering indexes require more disk space and maintenance overhead since they store additional columns. Ensure this is justified by frequent use of queries that benefit significantly.
Query Specificity: The chosen covering index should be tailored to specific query patterns to ensure it’s beneficial for most common operations.
Read-Write Trade-offs: While read performance is improved, write operations (like inserts and updates) are slowed down as the covering index must also be updated.
Choosing Between Covering and Plain Indexes
Use a plain secondary index when:
The query only needs data from columns already part of the index.
Read performance isn’t critical or there’s little benefit from additional columns in the index.
Choose a covering index when:
Multiple columns are needed for frequent queries, and minimizing read operations is crucial.
Data fetching patterns align well with specific index configurations.

#### Closing Summary

Covering indexes offer significant performance benefits by eliminating table lookups, but they come with increased storage and maintenance costs. The choice between a plain secondary index and a covering index depends on the specific query patterns and application requirements. Careful consideration of these factors ensures optimal database performance.

#### Common Mistakes/Follow-ups
Mistake 1: Assuming that adding columns to an index always improves performance, ignoring the overhead.

#### Follow-up
Question: "How do you determine if a covering index is appropriate for your specific use case?"

**Code:**

```sql
-- Example of creating a plain secondary index
CREATE INDEX idx_customer_id ON Customer (customer_id);

-- Example of creating a covering index
CREATE INDEX idx_customer_covering ON Customer (customer_id, name, email);
```

