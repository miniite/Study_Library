---
type: "practice"
difficulty: "Deep"
---

# What is a covering index (index-only scan) in Oracle and how do you design an index so the query never touches the table?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle SQL and Indexes](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A covering index, also known as an index-only scan, is a technique in Oracle that allows a database to retrieve all necessary data from an index without needing to access the underlying table. This can significantly improve query performance by reducing I/O operations and improving cache hit rates. To design such an index effectively, you need to understand both the structure of your queries and the schema of your tables.

#### Triaging the Question

The key here is understanding when a covering index would be beneficial and how to construct one that meets this criterion. The interviewer might probe into the performance benefits, trade-offs, and potential issues with over-indexing. We'll assume that the database contains a table employees with columns like emp_id, first_name, last_name, email, etc., and there are queries frequently targeting these fields.

#### First-Principles Solution
Identify Query Requirements: Determine which columns the query needs to retrieve.
Check Index Coverage: Ensure that an index exists on those columns, or combine multiple columns into a single covering index if possible.
Index Creation: Create an appropriate B-tree or bitmap index based on the column(s) required by the queries.
Test and Monitor: Verify the performance gains using EXPLAIN PLAN and monitor statistics to ensure the index is being used effectively.
Detailed Steps

Identify Query Requirements:

Suppose a common query involves selecting employees' names and emails:
SQL
SELECT first_name, last_name, email FROM employees WHERE emp_id = 123;
The columns first_name, last_name, and email need to be included in the index.

Check Index Coverage:

Create a covering index on the required columns:
SQL
CREATE INDEX idx_covering ON employees (emp_id, first_name, last_name, email);
Ensure that an index exists for all necessary queries or consider creating multiple indexes if needed.

Index Creation:

Oracle supports both B-tree and bitmap indexes. B-trees are good for most cases where data is evenly distributed, while bitmaps can be efficient for sparse data (e.g., boolean columns).
Use CREATE INDEX to create the covering index as shown above.

Test and Monitor:

Verify that the query uses the new index by running an EXPLAIN PLAN:
SQL
EXPLAIN PLAN FOR SELECT first_name, last_name, email FROM employees WHERE emp_id = 123;
Check the execution plan to ensure it indicates an index-only scan.
Monitor the performance over time and adjust as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Creating covering indexes can significantly boost query performance by reducing the need for table access. However, it’s important to balance this against potential overheads such as increased storage requirements and slower write operations due to index maintenance. Over-indexing can also lead to degraded performance if the indexes become too large or are not used efficiently.

#### Common mistakes
include:

Over-indexing leading to excessive storage usage.
Not considering the distribution of data, which might make bitmap indexes more appropriate for certain columns.
Failing to monitor and adjust indexes over time as query patterns change.

#### Follow-up
questions could explore scenarios where covering indexes wouldn't be suitable or how to handle complex queries with multiple conditions.

**Code:**

```sql
-- Example index creation
CREATE INDEX idx_covering ON employees (emp_id, first_name, last_name, email);
```

**Example input / output:**

```text
-- Before creating index
EXPLAIN PLAN FOR SELECT first_name, last_name, email FROM employees WHERE emp_id = 123;

-- After creating index
EXPLAIN PLAN FOR SELECT first_name, last_name, email FROM employees WHERE emp_id = 123;
```

**Data:**

```json
{
  "table": {
    "name": "employees",
    "columns": [
      {"name": "emp_id", "type": "INT"},
      {"name": "first_name", "type": "VARCHAR2(50)"},
      {"name": "last_name", "type": "VARCHAR2(50)"},
      {"name": "email", "type": "VARCHAR2(100)"}
    ]
  }
}
```

