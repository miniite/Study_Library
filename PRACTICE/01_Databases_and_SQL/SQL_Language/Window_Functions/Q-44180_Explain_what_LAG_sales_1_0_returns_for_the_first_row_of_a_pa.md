---
type: "practice"
difficulty: "Basic"
---

# Explain what LAG(sales, 1, 0) returns for the first row of a partition.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's break down the SQL window function LAG(sales, 1, 0) and understand its behavior when applied to the first row of a partition. The LAG() function is used to access the value from the previous row within the same partition. Here’s a step-by-step explanation:

TL;DR

The LAG(sales, 1, 0) function returns the value of sales from the previous row in the current partition for the first row. Since there is no previous row, it defaults to 0.

#### Triaging the Question
What does LAG do?: The LAG() function fetches a value from a specified offset position relative to the current row within a window.
Window context: We're focusing on the first row of a partition.
Default value: If there is no previous row, it returns a default value (in this case, 0).

#### First Principles Solution
Partitioning: First, identify how rows are grouped into partitions. For simplicity, let's assume our dataset contains sales data for different stores over time.
LAG function parameters:
sales: The column from which the value is fetched.
1: The offset position; it indicates that we want to fetch the value from the previous row (i.e., 1 row back).
0: The default value to return if there is no previous row.
Applying LAG to the First Row
Scenario: Consider a partition with multiple rows sorted by time.
First row consideration: For the first row in any partition, there are no preceding rows. Thus, according to the function definition, it should return the default value specified (0).

#### Closing Summary
+ Common Mistakes/Follow-Ups

The LAG(sales, 1, 0) function for the first row will always return 0 because there is no previous row from which to fetch a value. A common mistake might be misunderstanding that it should look at the next row instead of defaulting to the provided default value.

#### Example

Let's consider a simple example:

PLAINTEXT
+----+----------+
| id | sales    |
+----+----------+
| 1  | 10       |
| 2  | 15       |
| 3  | 20       |
+----+----------+

-- For the first row (id=1):
LAG(sales, 1, 0) => 0

This example clearly shows that for the first row in any partition, LAG returns 0.

**Comparison:**

```
id	sales	previous_sales
1	10	0
2	15	10
3	20	15
```

**Code:**

```sql
SELECT id,
       sales,
       LAG(sales, 1, 0) OVER (PARTITION BY some_column ORDER BY time_column) AS previous_sales
FROM your_table;
```

**Example input / output:**

```text
Input:
id, sales
1, 10
2, 15
3, 20

Output:
id, previous_sales
1, 0
2, 10
3, 15
```

**Data:**

```plaintext
+----+----------+-------------+
| id | sales    | previous_sales |
+----+----------+-------------+
| 1  | 10       | 0            |
| 2  | 15       | 10           |
| 3  | 20       | 15           |
+----+----------+-------------+
```

**Flow:**

1. First Row
2. (LAG(sales, 1, 0) returns 0)

