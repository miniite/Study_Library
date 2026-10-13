---
type: "practice"
difficulty: "Intermediate"
---

# How do you round a number down to the nearest 10 or 100 using math functions?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To round a number down to the nearest 10 or 100, we can use SQL's FLOOR function in combination with basic arithmetic operations. This approach is straightforward and works efficiently even for large datasets.

#### Triaging the Question

The core of this question is understanding how to manipulate numbers using SQL functions to achieve a specific rounding behavior. The interviewer might be probing your familiarity with common mathematical operations and their application within SQL queries. We need to ensure we handle both rounding down to 10s and 100s correctly.

#### First-Principles Solution

Let's start by understanding the basic operation:

Divide the number by the desired rounding factor: For rounding down to the nearest 10, divide by 10; for rounding down to the nearest 100, divide by 100.
Apply FLOOR function to round down to the nearest integer.
Multiply back by the rounding factor to get the final result.

Here's how you can implement this in SQL:

SQL
-- Example for rounding down to the nearest 10
SELECT FLOOR(number / 10) * 10 AS rounded_down_10 FROM your_table;

-- Example for rounding down to the nearest 100
SELECT FLOOR(number / 100) * 100 AS rounded_down_100 FROM your_table;

#### Closing Summary
and Common Mistakes

In summary, we used FLOOR combined with division and multiplication to achieve the desired rounding behavior. This method is efficient and works for any integer or floating-point number.

#### Common mistakes

Forgetting to use the * 10 or * 100 after applying FLOOR.
Not checking if your input values are non-negative, as division by zero will throw an error.
Assuming that SQL's ROUND function can be used for this purpose; remember ROUND performs standard rounding (to the nearest), not always down.
Sample Code

Here is a sample code snippet to demonstrate both operations:

SQL
-- Example data
CREATE TABLE numbers (
    id INT PRIMARY KEY,
    number DECIMAL(10, 2)
);

INSERT INTO numbers (id, number) VALUES
(1, 54.37),
(2, 689.23),
(3, 1234.56);
Worked Example

Let's see how this works with the sample data:

SQL
SELECT id,
       number,
       FLOOR(number / 10) * 10 AS rounded_down_10,
       FLOOR(number / 100) * 100 AS rounded_down_100
FROM numbers;

Output:

id	number	rounded_down_10	rounded_down_100
1	54.37	50	0
2	689.23	680	600
3	1234.56	1230	1200
Closing Thoughts

This approach ensures that any number, regardless of its value or scale, is rounded down to the nearest multiple of 10 or 100 using fundamental SQL operations. Always double-check your assumptions and handle edge cases appropriately.

**Code:**

```sql
-- Rounding down to the nearest 10
SELECT FLOOR(number / 10) * 10 AS rounded_down_10 FROM numbers;

-- Rounding down to the nearest 100
SELECT FLOOR(number / 100) * 100 AS rounded_down_100 FROM numbers;
```

**Data:**

```sql
CREATE TABLE numbers (
    id INT PRIMARY KEY,
    number DECIMAL(10, 2)
);

INSERT INTO numbers (id, number) VALUES
(1, 54.37),
(2, 689.23),
(3, 1234.56);
```

