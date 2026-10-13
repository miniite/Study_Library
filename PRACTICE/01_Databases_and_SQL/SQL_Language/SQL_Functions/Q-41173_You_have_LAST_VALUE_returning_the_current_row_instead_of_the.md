---
type: "practice"
difficulty: "Deep"
---

# You have LAST_VALUE() returning the current row instead of the partition's last value. Explain the frame issue and the correct frame specification to fix it.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UDF Performance](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with SQL functions like LAST_VALUE, understanding how window frames are specified is crucial. In this scenario, you've encountered an issue where LAST_VALUE() returns the current row instead of the last value in its partition. This problem arises due to incorrect frame specification.

The core concept here is that LAST_VALUE() requires a proper frame definition using the ROWS BETWEEN clause to ensure it correctly identifies and returns the desired row within the partition. Let's break down the solution from first principles.

**Comparison:**

| id | column1 | column2 | partition_column | order_column |
|----|---------|---------|------------------|--------------|
| 1  | A       | 30      | X                | 1            |
| 2  | B       | 40      | X                | 2            |
| 3  | C       | 50      | X                | 3            |

**Code:**

```

-- Incorrect use of LAST_VALUE() SELECT column1, LAST_VALUE(column2) OVER (PARTITION BY partition_column ORDER BY order_column) FROM your_table;

CODE
Copy
This query does not specify a frame, which can lead to unexpected results. To fix this issue, we need to define the correct frame specification.

```sql
-- Correct use of LAST_VALUE() with proper frame specification
SELECT
    column1,
    LAST_VALUE(column2) OVER (
        PARTITION BY partition_column
        ORDER BY order_column
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW -- This is the key correction
    ) AS last_value_of_partition
FROM
    your_table;

The ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW clause ensures that LAST_VALUE() looks at all rows in the partition up to and including the current row, thereby correctly returning the last value.

SQL
Copy
-- Example Table Schema for illustration
CREATE TABLE your_table (
    id INT,
    column1 VARCHAR(255),
    column2 INT,
    partition_column VARCHAR(255),
    order_column INT
);
```

**Example input / output:**

```text
-- Input query with correct frame specification
SELECT
    column1,
    LAST_VALUE(column2) OVER (
        PARTITION BY partition_column
        ORDER BY order_column
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS last_value_of_partition
FROM
    your_table;

-- Expected output
+--------+----------------+
| column1| last_value_of_partition |
+--------+----------------+
| A      | 50                  |
| B      | 50                  |
| C      | 50                  |
+--------+----------------+
```

**Data:**

```json
[
    {"id": 1, "column1": "A", "column2": 30, "partition_column": "X", "order_column": 1},
    {"id": 2, "column1": "B", "column2": 40, "partition_column": "X", "order_column": 2},
    {"id": 3, "column1": "C", "column2": 50, "partition_column": "X", "order_column": 3}
]
```

**Flow:**

1. Query with Correct FrameSpecification
2. PARTITION BYpartition_column
3. ORDER BY order_column
4. ROWS BETWEENUNBOUNDED PRECEDINGAND CURRENT ROW
5. LAST_VALUE(column2)Returned Correctly

