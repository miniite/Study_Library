---
type: "practice"
difficulty: "Basic"
---

# How would you write validation queries to check that a numeric column has no negative or NULL values before publishing a table?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a numeric column in a database contains only non-negative and non-NULL values, we need to perform validation checks on this data. These checks are crucial because they guarantee the integrity of our data before it is published. Let’s break down the steps required to write such validation queries.

Firstly, when dealing with numeric columns, we typically want to ensure that all values are either positive or zero and that no value is NULL. We can achieve this by writing SQL queries that will validate these conditions.

**Code:**

```sql
-- Example table structure
CREATE TABLE example_table (
    id INT,
    numeric_column DECIMAL(10, 2)
);
```

**Example input / output:**

```text
-- Sample data for validation
INSERT INTO example_table (id, numeric_column) VALUES
(1, -1.5), (2, NULL), (3, 4.2), (4, 0), (5, 7.8);

-- Validation queries
SELECT COUNT(*) AS negative_values_count FROM example_table WHERE numeric_column < 0;
-- Expected output: negative_values_count = 1

SELECT COUNT(*) AS null_values_count FROM example_table WHERE numeric_column IS NULL;
-- Expected output: null_values_count = 1
```

