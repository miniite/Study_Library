---
type: "practice"
difficulty: "Basic"
---

# How would you store a metric value safely as a numeric type, and why is storing it as text a problem for aggregation?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store a metric value safely as a numeric type in SQL, we need to ensure that the data can be accurately represented and manipulated mathematically. Storing metrics as text introduces several issues, particularly when performing aggregation operations like summing or averaging.

First, let's consider why storing metric values as text is problematic:

Inaccurate Aggregation: When aggregating numeric data, such as calculating a sum or average, the result can be incorrect if stored as text. For example, adding "20" and "30" as strings would give you "2030" instead of 50.
Performance Overhead: Text fields generally have higher storage overhead compared to numeric fields due to additional metadata required for string representation.
Loss of Precision: In some cases, storing data as text can lead to loss of precision when converting back and forth between numeric and textual formats.

Now, let's explore how we would store a metric value safely in SQL:

Data Type Selection:

Use an appropriate numeric data type such as INT, BIGINT, or DECIMAL (for floating-point numbers).
For example, if the metric is always an integer and can be large, use BIGINT.

Insertion Process:

SQL
INSERT INTO metrics_table(metric_name, timestamp, value) VALUES('metric1', NOW(), 100);

Query for Aggregation:

To aggregate the metric over a time period, you can use SQL functions like SUM() and AVG().
SQL
SELECT metric_name, SUM(value) AS total_value, AVG(value) AS average_value
FROM metrics_table
WHERE timestamp BETWEEN '2023-01-01' AND NOW()
GROUP BY metric_name;

Error Handling:

Ensure that any input to the database is validated before insertion to prevent data integrity issues.
Use constraints like CHECK to enforce valid numeric values.

#### Closing Summary
Storing metrics as text can lead to inaccurate calculations and performance overhead, making it a poor choice for aggregation operations in SQL. Using appropriate numeric types ensures accurate data representation and efficient querying.

#### Common Mistakes

Incorrectly storing metric values as text.
Failing to validate input before insertion into the database.
Not using proper constraints or indexes to optimize query performance.

**Comparison:**

| Column Name | Type     |
|-------------|----------|
| id          | INT      |
| metric_name | VARCHAR  |
| timestamp   | TIMESTAMP|
| value       | BIGINT   |

**Code:**

```sql
-- Example of inserting a metric value
INSERT INTO metrics_table(metric_name, timestamp, value) VALUES('metric1', NOW(), 100);

-- Example query for aggregation
SELECT metric_name, SUM(value) AS total_value, AVG(value) AS average_value
FROM metrics_table
WHERE timestamp BETWEEN '2023-01-01' AND NOW()
GROUP BY metric_name;
```

**Example input / output:**

```text
-- Input
INSERT INTO metrics_table(metric_name, timestamp, value) VALUES('metric1', NOW(), 100);

-- Output (example)
+---------+--------------+---------------------+----------+
| id      | metric_name  | timestamp           | value    |
+---------+--------------+---------------------+----------+
| 1       | metric1      | 2023-06-08 14:59:17 | 100      |
+---------+--------------+---------------------+----------+
```

**Data:**

```json
{
  "metrics_table": {
    "columns": [
      {"name": "id", "type": "INT"},
      {"name": "metric_name", "type": "VARCHAR(255)"},
      {"name": "timestamp", "type": "TIMESTAMP"},
      {"name": "value", "type": "BIGINT"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Insert Metric Value
4. Is the value a number?
5. Use numeric type
6. Convert to text and store
7. Perform aggregation
8. Convert back for aggregation,if needed

