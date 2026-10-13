---
type: "practice"
difficulty: "Intermediate"
---

# Your composite index has an unused trailing column bloating it — how do you safely prove it's unused before dropping it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a composite index in a database, especially one that includes an unused trailing column, it’s crucial to verify if that trailing column is indeed not being used before considering its removal. This ensures we don’t accidentally degrade query performance or lose valuable data access patterns. For this task, let's assume you're working with PostgreSQL, as it supports advanced indexing features.

The key idea here is to use a combination of EXPLAIN and aggregate functions to determine if the trailing column is being used in queries. We'll walk through the process step-by-step from first principles.

**Code:**

```sql
-- Example table schema
CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    department_id INT,
    hire_date DATE
);

-- Composite index with an unused trailing column 'last_name'
CREATE INDEX idx_department_hire_date_last_name ON employees(department_id, hire_date, last_name);
```

