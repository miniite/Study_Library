---
type: "practice"
difficulty: "Deep"
---

# A slow query returns few rows but reads many index pages. What does a high index-tuples-read-to-rows-returned ratio imply about your index design?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database indexing and query optimization, a high index-tuples-read-to-rows-returned (I/R) ratio indicates that while your query is retrieving few rows from the table, it is reading many index pages. This can imply several potential issues with your index design or the way queries are written. Let's break this down step by step.

#### Triage

The question asks us to interpret a specific performance metric: the index-tuples-read-to-rows-returned ratio. The key here is understanding why such a high ratio might occur and what it tells us about our index design or query strategy.

#### Assumptions
We are dealing with an indexed table.
The query returns few rows but reads many index pages.
Indexes are used to improve query performance, so the focus should be on why they aren't as effective as expected.

#### First-Principles Solution
1. Understanding the Metric
index-tuples-read-to-rows-returned is a ratio that tells us how many index entries (tuples) are read from an index for each row returned by a query.
A high I/R ratio means that the database reads more index pages than necessary to return only a few rows.
2. Possible Causes
Index Overuse: The query is unnecessarily using the index, which might mean it's not leveraging the index effectively or at all.
Inefficient Query Conditions: The conditions in the WHERE clause are not selective enough, causing more index entries to be checked than necessary.
Non-Selective Indexes: The indexes chosen are not very effective for the query, leading to a large number of index accesses even though only few rows match.
3. Diagnosis and Fixes

Review Query Conditions:

SQL
SELECT * FROM employees WHERE salary > 1000 AND department_id = 2;

Ensure that the conditions in the query are as selective as possible. For example, if department_id is not very selective (many rows have this value), consider adding a more specific condition.

Choose Appropriate Indexes:

SQL
CREATE INDEX idx_salary_department ON employees(salary, department_id);

#### Cons
ider creating composite indexes that cover the conditions used in the query. In our example, an index on (department_id, salary) might be more effective than a separate index on salary.

Index Maintenance: Ensure that your database is properly maintained to keep statistics up-to-date, which helps the query optimizer make better decisions.

4. Architectural Considerations
Query Planning: The cost-based query planner might not always choose the optimal execution plan, especially in complex queries.
Index Design Patterns: Follow best practices for index design. For example, avoid creating overly broad indexes that can degrade performance due to increased write overhead and I/O usage.

#### Closing Summary

A high index-tuples-read-to-rows-returned ratio indicates inefficient use of the index or an ineffective query condition. This can be addressed by refining query conditions, selecting appropriate indexes, and ensuring proper maintenance practices. Understanding these issues helps in making informed decisions to optimize database performance.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the I/R ratio altogether.

#### Follow-Up
What other metrics would you look at when diagnosing query performance?
How do you balance between writing efficient queries and maintaining good indexing practices in a large, complex database environment?

**Comparison:**

```
Index	Tuples Read	Rows Returned
idx_department	1000	5
PLAINTEXT
Copy
High I/R Ratio Explanation:
Index reads more tuples than necessary to return a small number of rows.
```

**Code:**

```sql
-- Example of a non-selective WHERE clause
SELECT * FROM employees WHERE department_id = 2;

-- Example of an index that might be inefficient
CREATE INDEX idx_department ON employees(department_id);
```

**Example input / output:**

```text
Input:
SELECT * FROM employees WHERE department_id = 2;

Output (with high I/R ratio):
- Read many index pages for few matching rows.
```

**Data:**

```json
{
  "table": {
    "name": "employees",
    "columns": ["id", "name", "salary", "department_id"]
  },
  "indexes": [
    {
      "name": "idx_department",
      "columns": ["department_id"]
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is the query selective enough?
5. Review and refine WHEREclause
6. Check if index is appropriate
7. Create or adjust indexes
8. Maintain database statistics
9. Optimized query execution

