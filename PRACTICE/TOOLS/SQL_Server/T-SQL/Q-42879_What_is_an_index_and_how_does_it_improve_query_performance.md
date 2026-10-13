---
type: "practice"
difficulty: "Basic"
---

# What is an index and how does it improve query performance?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In T-SQL, an index is a data structure that improves the speed of data retrieval operations on a database table. Essentially, it's like having a map to quickly find specific pieces of information in a large book. Let’s break down what an index does and how it works.

#### Triaging the Question

The question is asking for an explanation of the concept of an index in T-SQL and its role in enhancing query performance. The interviewer is probing your understanding of fundamental database concepts and how to optimize queries by using indexes effectively.

#### First-Principles Solution

An index in T-SQL is a structure that enables faster data retrieval from a table. When a column or set of columns is indexed, the database engine can use this index to find specific records more efficiently than if it had to scan every row. Indexes are created on one or more columns and can significantly speed up SELECT, INSERT, UPDATE, and DELETE operations.

Key Concepts:
Types of Indexes: Common types include B-tree (default), nonclustered, unique, and covering indexes.
How Indexes Work:
An index creates a separate data structure that contains pointers to the actual rows in your table.
When you query a column or set of columns used in an index, the database engine can quickly locate the relevant data by following these pointers.
Full Solution

Indexes improve query performance by reducing the number of disk I/O operations required to retrieve data. Here’s how it works step-by-step:

Creating an Index:

SQL
CREATE INDEX idx_name ON table_name (column1, column2);

This command creates a B-tree index on table_name using columns column1 and column2.

Using an Index in Queries: When you run a query that involves these columns, the database engine can use the index to quickly locate the relevant rows.

Query Plan Analysis: Use the EXPLAIN command or SET SHOWPLAN_ALL ON to see how the optimizer uses indexes.

SQL
SET SHOWPLAN_ALL ON;
SELECT * FROM employees WHERE department = 'Sales';

Index Maintenance: Indexes need maintenance; they can become fragmented over time, which impacts performance. Tools like REBUILD and REORGANIZE help in maintaining index efficiency.

SQL
ALTER INDEX idx_name ON table_name REBUILD;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Indexes are powerful tools for optimizing queries but come with trade-offs:

Space Overhead: Indexes take up additional storage space, which can slow down INSERT and DELETE operations.
Maintenance Costs: Frequent updates to indexes can impact performance.

#### Common mistakes
include over-indexing (adding too many indexes) or not using appropriate indexing strategies. For example, adding an index on a column that is rarely used in queries will not provide much benefit but can still consume resources.

#### Follow-Up
When and Where Should You Use Indexes?

When deciding to use an index, consider factors like the selectivity of columns (how many distinct values), the frequency of updates, and the presence of foreign key relationships. Use indexes on columns that are frequently used in WHERE clauses or as part of joins.

**Comparison:**

```
id	name	department
1	John Doe	Sales
2	Jane Smith	Engineering
```

**Code:**

```sql
CREATE INDEX idx_name ON employees (department);
ALTER INDEX idx_name ON employees REBUILD;
```

**Example input / output:**

```text
-- Before REBUILD
SET SHOWPLAN_ALL ON;
SELECT * FROM employees WHERE department = 'Sales';
-- After REBUILD
ALTER INDEX idx_name ON employees REBUILD;
SET SHOWPLAN_ALL ON;
SELECT * FROM employees WHERE department = 'Sales';
```

**Data:**

```json
{
  "employees": [
    { "id": 1, "name": "John Doe", "department": "Sales" },
    { "id": 2, "name": "Jane Smith", "department": "Engineering" }
  ]
}
```

**Flow:**

1. No
2. Query Execution
3. (Does Index Exist?)
4. Use Index for Retrieval
5. Full Table Scan
6. (Rebuild Index if Necessary)
7. Continue Query Execution

