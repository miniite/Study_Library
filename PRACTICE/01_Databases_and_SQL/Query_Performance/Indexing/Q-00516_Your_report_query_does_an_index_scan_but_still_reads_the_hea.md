---
type: "practice"
difficulty: "Intermediate"
---

# Your report query does an index scan but still reads the heap for every row — how would you turn it into an index-only scan in PostgreSQL?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Covering Indexes and Index Strategy for Sharding](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/Covering_Indexes_and_Index_Strategy_for_Sharding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand why a query that should be using an index is still reading from the heap. The key here is turning a regular index scan into an index-only scan (IOT) in PostgreSQL. This means minimizing or eliminating heap accesses by leveraging additional columns stored in the index.

TL;DR

We will examine the current query and schema, identify the missing columns required to make the index-only scan possible, and create a composite or covering index that includes those necessary fields. Then, we'll modify the query to take advantage of this new index.

#### Triaging the Question

The question is asking how to optimize a query in PostgreSQL by converting an existing index scan into an index-only scan (IOT). The core issue lies in ensuring that all required data can be fetched directly from the index without accessing the heap. To do this, we need to understand:

What columns are currently being accessed?
Are these columns already included in or can they be added to the existing index?

#### First-Principles Solution
Step 1: Analyze the Current Query and Schema

First, let's look at the query that is performing an index scan but reading from the heap:

SQL
SELECT column1, column2, column3 FROM table_name WHERE condition;

We need to understand which columns are being accessed in the SELECT clause. If any of these columns are not part of the current index, then the database will read from the heap.

Step 2: Determine if a Covering or Composite Index is Needed

A covering index includes all the columns needed by the query, so they can be fetched entirely from the index. A composite index further optimizes the situation by allowing the query to use an index on multiple columns in the specified order.

Assuming column1, column2, and column3 are part of our query, we need to check if any of them are missing from the existing index:

SQL
CREATE INDEX idx_name ON table_name (column1, column2);

If column3 is not included in this index, it means that every time a row matches the condition and column3 needs to be fetched, PostgreSQL will have to read from the heap.

Step 3: Create a Covering or Composite Index

To ensure an index-only scan, we should create a covering index:

SQL
CREATE INDEX idx_name_covering ON table_name (column1, column2) INCLUDE (column3);

This INCLUDE keyword allows PostgreSQL to store additional columns in the index, reducing heap accesses.

Step 4: Modify the Query if Necessary

In this case, since we've already covered all necessary columns in the index, no changes are needed to the query itself. However, it's good practice to ensure that your query aligns well with the created indexes:

SQL
SELECT column1, column2, column3 FROM table_name WHERE condition;

#### Closing Summary
and Common Mistakes/Follow-Ups

By creating a covering index, we can reduce or eliminate heap accesses for this specific query. This not only speeds up the execution but also minimizes I/O operations, which is crucial for performance optimization.

#### Common Mistakes

Not Including All Required Columns: If any column in the SELECT clause is missing from the index, an index scan will still result in heap accesses.
Ignoring Composite Indexes: A composite index can further optimize queries that access columns in a specific order.
Triangulating Follow-Ups

The interviewer may ask:

How does PostgreSQL decide which index to use?
What are other ways to improve query performance beyond indexing?
How would you handle the scenario where data distribution is uneven?

These follow-ups encourage deeper discussion on database optimization strategies and reinforce your understanding of how PostgreSQL operates.

**Code:**

```sql
CREATE INDEX idx_name_covering ON table_name (column1, column2) INCLUDE (column3);
```

