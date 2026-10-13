---
type: "practice"
difficulty: "Expert"
---

# Explain how a window function's frame boundary of 'GROUPS' differs from ROWS and RANGE, and give a case where GROUPS is the right choice.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the nuances between ROWS, RANGE, and GROUPS in window functions, let's first lay out what each frame boundary does:

ROWS defines a frame based on the number of rows relative to the current row.
RANGE is similar but uses a range of values from the current row based on an order-by expression.
GROUPS defines a frame by splitting the data into contiguous groups, where each group starts and ends with specific conditions.

Essentially, GROUPS allows us to define window frames that are not just sequential rows or ranges but can be grouped by distinct segments in the data, which is particularly useful when dealing with complex data segmentation needs. We'll explore this through a case study.

**Comparison:**

|id |value  |category |cumulative_value|
|---|-------|---------|----------------|
|1  |100    |A        |100             |
|2  |-50    |A        |50              |
|3  |75     |B        |75              |
|4  |-25    |A        |25              |

**Code:**

```sql
-- Example of using GROUPS in a SQL query
SELECT
    id,
    value,
    SUM(value) OVER (
        PARTITION BY category -- Define the groups by 'category'
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_value
FROM transactions;
```

**Example input / output:**

```sql
-- Sample input data
WITH transactions AS (
    SELECT
        id,
        value,
        category,
        timestamp
    FROM (VALUES
        (1, 100, 'A', '2023-10-01'),
        (2, -50, 'A', '2023-10-02'),
        (3, 75, 'B', '2023-10-03'),
        (4, -25, 'A', '2023-10-04')
    ) AS t(id, value, category, timestamp)
)

-- Query using GROUPS
SELECT
    id,
    value,
    category,
    SUM(value) OVER (
        PARTITION BY category -- Define the groups by 'category'
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_value
FROM transactions;
id	value	category	cumulative_value
1	100	A	100
2	-50	A	50
3	75	B	75
4	-25	A	25
```

**Data:**

```json
{
  "transactions": [
    {"id": 1, "value": 100, "category": "A", "timestamp": "2023-10-01"},
    {"id": 2, "value": -50, "category": "A", "timestamp": "2023-10-02"},
    {"id": 3, "value": 75, "category": "B", "timestamp": "2023-10-03"},
    {"id": 4, "value": -25, "category": "A", "timestamp": "2023-10-04"}
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. Start
4. Is the data grouped by aspecific category?
5. Define groups using`PARTITION BY`
6. Order rows based ontimestamp (or anotherorder-by column)
7. Use 'ROWS BETWEENUNBOUNDED PRECEDINGAND CURRENT ROW' to defineframe?
8. Calculate cumulative sum overdefined frame boundaries
9. End

