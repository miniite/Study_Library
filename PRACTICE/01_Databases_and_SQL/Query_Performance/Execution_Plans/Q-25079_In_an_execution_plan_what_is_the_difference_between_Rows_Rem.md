---
type: "practice"
difficulty: "Deep"
---

# In an execution plan what is the difference between 'Rows Removed by Filter' and rows returned?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database execution plan, 'Rows Removed by Filter' represents the number of records that were filtered out during the execution of a SQL query due to predicate conditions. On the other hand, "rows returned" refers to the total number of rows that matched the conditions and were outputted by the query. Essentially, the difference highlights the filtering process in more detail.

To understand this distinction better, let's break down the concepts from first principles:

#### Triage
Question: What is the difference between 'Rows Removed by Filter' and "rows returned"?
Context: In a database execution plan, these terms are used to describe different stages of data processing.

#### Assumptions
We assume you have basic knowledge of SQL queries and how they are executed in a database.

#### First Principles Solution

Understanding Execution Plans:

An execution plan shows the steps a database takes to retrieve or manipulate data based on an SQL query.
Each step is represented as an operator, which could be a table scan, index lookup, join operation, etc.

'Rows Removed by Filter':

This term typically appears in operations like WHERE clauses, joins, and filters within other operators (like Hash Join or Merge Join).
It indicates how many rows were discarded due to a condition not being met.
For example, if you have a query filtering on a column value WHERE column = 'value', the number of rows removed by this filter will be those that do not match 'value'.

"Rows Returned":

This refers to the total number of rows outputted from an operator or step.
It includes all rows that matched any condition, before applying filters.
For a SELECT operation, it would be the final number of rows returned by the query.

#### Example

#### Cons
ider a query like this:

SQL
SELECT * FROM customers WHERE city = 'New York';
Execution Plan:
CODE
Filter: [city = 'New York'] -> Rows Removed by Filter: 150
Table Scan: customers -> Rows Returned: 2000

In this example:

Rows Returned is 2000, meaning that the table scan returned 2000 rows.
Rows Removed by Filter is 150, indicating that 150 of those rows did not meet the condition city = 'New York'.

#### Closing Summary

Understanding these terms helps in analyzing and optimizing SQL queries. 'Rows Removed by Filter' gives insight into how filters are applied and can help identify inefficient conditions or missing indexes.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring the difference between rows returned and rows removed.

#### Follow-up
Ask about the impact of filtering early in a query (e.g., using an index) versus late.

Mistake: Assuming that minimizing rows returned is always beneficial.

#### Follow-up
Discuss scenarios where returning more rows can be advantageous, such as during aggregation or when multiple filters are applied.

**Code:**

```sql
-- Example query
SELECT * FROM customers WHERE city = 'New York';

-- Execution Plan (hypothetical)
Filter: [city = 'New York'] -> Rows Removed by Filter: 150
Table Scan: customers -> Rows Returned: 2000
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT * FROM customers WHERE city = 'New York';

Output:

CODE
Copy
Filter: [city = 'New York'] -> Rows Removed by Filter: 150
Table Scan: customers -> Rows Returned: 2000
```

**Data:**

```json
{
  "rows_returned": 2000,
  "rows_removed_by_filter": 150
}
```

**Flow:**

1. Query Execution Plan
2. Rows Returned
3. Rows Removed by Filter
4. Filter Step
5. Table Scan

