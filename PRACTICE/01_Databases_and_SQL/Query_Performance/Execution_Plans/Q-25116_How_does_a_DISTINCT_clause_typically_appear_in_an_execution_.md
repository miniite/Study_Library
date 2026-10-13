---
type: "practice"
difficulty: "Intermediate"
---

# How does a DISTINCT clause typically appear in an execution plan and can it be expensive?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: When dealing with a DISTINCT clause, its presence in an execution plan is often to filter out duplicate rows from the result set. While DISTINCT operations are generally straightforward, they can be costly depending on the query and database system used.

#### Triage
The question probes your understanding of how the DISTINCT operation affects the execution plan and whether it could lead to performance issues. Key assumptions include familiarity with basic SQL concepts, knowledge of execution plans, and an understanding of how databases process distinct values.

#### Solution

Understanding DISTINCT: A DISTINCT clause is used in SQL to eliminate duplicate rows from the result set. When a query includes SELECT DISTINCT, the database needs to identify unique combinations of column values and return only those rows.
Execution Plan Impact: In an execution plan, you might see operations like "Sort" or "Distinct Operator" associated with the presence of DISTINCT. The exact representation can vary by database system (e.g., PostgreSQL shows a "Hash Aggregate" for distinct processing).
Cost Considerations:
Sorting Method: Sorting all rows first and then filtering out duplicates is one common method, which can be costly if the table has many rows.
Using Indexes: If an appropriate index exists, the database might use it to filter duplicates more efficiently. However, not every DISTINCT query can leverage indexes effectively.
Memory Constraints: Hash-based methods (like PostgreSQL's "Hash Aggregate") require significant memory allocation and can fail under high memory pressure.

#### Example
Execution Plan:

PLAINTEXT
Sort
  Distinct Operator
    Filter
      Table Scan (table: employees, columns: id, name)
Performance Analysis:
The Table Scan operation retrieves all rows from the table.
The Filter condition ensures that only relevant rows are processed next.
The Distinct Operator sorts and filters these rows to eliminate duplicates.

#### Closing Summary
While DISTINCT clauses are useful for ensuring unique results, they can introduce performance overhead due to additional sorting and filtering steps. Efficient use of indexes and careful query optimization can mitigate some of the costs associated with distinct processing.

#### Common Mistakes

Overlooking the impact of DISTINCT on large datasets.
Assuming that every DISTINCT operation will be slow, without considering database optimizations and indexing strategies.
Ignoring the potential for memory limitations when using hash-based distinct operations.

**Comparison:**

| Operation   | Description                |
|-------------|----------------------------|
| Table Scan  | Retrieves all rows         |
| Filter      | Applies conditions         |
| Distinct    | Filters out duplicates     |
| Sort        | Sorts the result set       |

**Code:**

```sql
SELECT DISTINCT id, name FROM employees;
```

**Example input / output:**

```text
Input:
SELECT DISTINCT id, name FROM employees;

Output:
+----+----------+
| id | name     |
+----+----------+
| 1  | Alice    |
| 2  | Bob      |
| 4  | Charlie  |
+----+----------+
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    {"id": 3, "name": "Alice"},
    {"id": 4, "name": "Charlie"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Table Scan
4. Index Available?
5. Filter Conditions
6. Distinct Operator
7. Sort and Distinct
8. Return Unique Rows

