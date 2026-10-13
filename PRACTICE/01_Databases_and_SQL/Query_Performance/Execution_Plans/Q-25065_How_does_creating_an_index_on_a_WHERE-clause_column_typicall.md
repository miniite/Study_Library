---
type: "practice"
difficulty: "Basic"
---

# How does creating an index on a WHERE-clause column typically change the execution plan?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you create an index on a column used in a WHERE clause, it can significantly alter the execution plan of your SQL query. In essence, this change helps the database engine to optimize its search process by reducing the amount of data it needs to scan.

To understand how an index affects the execution plan, we need to start from first principles: what is an execution plan and why do we care about indexes? An execution plan represents the series of steps a database system takes to retrieve or manipulate data. The goal is always to find the most efficient way to execute the query. By creating an index on a column used in the WHERE clause, you’re essentially telling the database engine which parts of the table it should consider first, thereby speeding up the process.

Let's walk through this step-by-step:

Understanding Indexes: An index is like a dictionary that allows you to quickly find information without having to read each page in a book from cover to cover. In databases, an index on a WHERE clause column means the database can locate rows more efficiently.
Execution Plan Basics: A basic execution plan starts with identifying which columns are needed and then determining how best to retrieve those columns. Without an index, the database might have to perform a full table scan (reading every row), a sort operation, or a join. With an index, the database can immediately jump to the relevant rows.
Impact on Query Execution: When you create an index on a WHERE clause column, the database engine will often use that index as a primary means of accessing the data. This typically leads to faster query execution because it reduces the need for full table scans and can allow the database to leverage techniques like seek operations instead.

For example, consider a simple SQL query:

SQL
SELECT * FROM employees WHERE department_id = 10;

Without an index on department_id, the database might perform a full table scan. However, with an index on department_id, it can quickly locate all rows where department_id equals 10.

Here is what the execution plan might look like in a tool like SQL Server Management Studio (SSMS):

SQL
|--Clustered Index Scan(OBJECT: [employees[PK_employees]])

Becomes:

SQL
|--Index Seek(OBJECT: [employees_idx_department_id], SEEK Conditions: ([employees.department_id] = 10))

This change from a Clustered Index Scan to an Index Seek is indicative of the performance improvement.

It's important to note that while indexes can speed up query execution, they also come with maintenance costs. They take up storage space and require regular updates (inserts, deletes, updates) which can impact write performance. Therefore, it’s crucial to create indexes only where necessary and to carefully consider their potential impact on the overall system.

**Comparison:**

| Operation                 | Details                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| Without Index             | Clustered Index Scan on employees table                                  |
| With Index                | Index Seek using employees_idx_department_id index                       |

**Code:**

```sql
-- Example SQL query without an index
SELECT * FROM employees WHERE department_id = 10;

-- Execution plan without an index (simplified)
|--Clustered Index Scan(OBJECT: [employees[PK_employees]])

-- Example execution plan with a properly created index
-- Assuming 'CREATE INDEX idx_department_id ON employees(department_id)' has been executed
SELECT * FROM employees WHERE department_id = 10;

-- Simplified execution plan after creating the index
|--Index Seek(OBJECT: [employees_idx_department_id], SEEK Conditions: ([employees.department_id] = 10))
```

**Example input / output:**

```text
Input Query:
SELECT * FROM employees WHERE department_id = 10;

Without Index (Full Table Scan):
Execution Plan:
|--Clustered Index Scan(OBJECT: [employees[PK_employees]])

With Index (Index Seek):
Execution Plan:
|--Index Seek(OBJECT: [employees_idx_department_id], SEEK Conditions: ([employees.department_id] = 10))
```

**Data:**

```json
{
  "table": {
    "name": "employees",
    "columns": [
      {"name": "employee_id", "type": "INT"},
      {"name": "first_name", "type": "VARCHAR(50)"},
      {"name": "last_name", "type": "VARCHAR(50)"},
      {"name": "department_id", "type": "INT"}
    ]
  },
  "index": {
    "name": "employees_idx_department_id",
    "columns": ["department_id"]
  }
}
```

**Flow:**

1. No
2. Yes
3. SQL Query
4. Is an index on 'department_id'created?
5. Full Table Scan
6. Index Seek
7. Slow Execution
8. Fast Execution

