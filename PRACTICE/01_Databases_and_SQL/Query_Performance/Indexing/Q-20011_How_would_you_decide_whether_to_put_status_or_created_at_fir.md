---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide whether to put status or created_at first in a composite index for WHERE status = 'open' ORDER BY created_at?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding how to order columns in a composite index for queries involving WHERE status = 'open' ORDER BY created_at, you need to consider several factors including query frequency, data distribution, and performance metrics. The key idea is that the most selective column (the one with fewer possible values) should come first in the index.

#### Triage

The question probes your understanding of how composite indexes work, the importance of order within an index, and the impact on query performance. We will assume:

The table has a high number of rows.
status is either 'open' or 'closed'.
created_at is a timestamp with many unique values.
Step-by-Step Solution
Identify Selectivity: status is the most selective column since it only has two possible values, making it a good candidate for first position.
Order of Columns: Placing status first will allow the database to quickly filter rows where status = 'open'. Then, created_at can be used to order these filtered results efficiently.

#### Cons
iderations:
Filtering Efficiency: By filtering based on a selective column first (status), we reduce the number of rows that need to be scanned, leading to faster query execution.
Ordering Cost: Using created_at as the second column ensures that only relevant results are considered for sorting, keeping the ordering cost low.
Full Solution

Given the high selectivity and the nature of the query, we should create a composite index on (status, created_at). This means:

The database will first use the index to quickly find all rows where status = 'open'.
Then it will sort these results based on created_at.

#### Closing Summary

Placing status before created_at in the composite index is optimal for this query. It leverages the high selectivity of status, reducing the number of rows to be sorted, and ensuring that only relevant results are processed.

#### Common Mistakes
& Follow-Ups
Common Mistake: Placing created_at first can lead to inefficient queries if status has many possible values.

#### Follow-Up
Consider additional constraints or indexes for other common query patterns. For example, if there are frequently queried status values other than 'open', adding separate indexes could be beneficial.

**Code:**

```sql
CREATE INDEX idx_open_order ON my_table (status, created_at);
```

**Example input / output:**

```

Input:

Query: SELECT * FROM my_table WHERE status = 'open' ORDER BY created_at;

Output:

Database uses the composite index (status, created_at).
Filters rows where status = 'open'.
Sorts filtered results by created_at.

Dry-run:

PLAINTEXT
Copy
Step 1: Index lookup for status='open'
Step 2: Sort result set based on created_at
```

**Flow:**

1. Yes
2. Yes
3. No
4. Filter status = 'open'
5. Use index (status, created_at)?
6. Sort by created_at?
7. Return results
8. Return all rows and sort

