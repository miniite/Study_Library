---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd expand a range table of (subscription_id, start_date, end_date) into daily active rows in a warehouse using a generated date sequence, and how to avoid materializing a huge cross product.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expanding Date Ranges](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expand a range table of (subscription_id, start_date, end_date) into daily active rows in a data warehouse, we need to generate a date sequence between start_date and end_date for each subscription and then join this sequence with the original subscriptions. This approach avoids materializing a huge cross product by dynamically generating dates on demand.

The core idea is to use a combination of window functions and common table expressions (CTEs) or subqueries to efficiently generate the date sequences and perform the necessary joins without pre-computing all possible combinations, which can be memory-intensive and slow for large datasets.

#### Triaging the Question
Understanding the Requirement: We need to expand each subscription's range into daily rows.
Avoiding Materialization: The goal is to avoid creating a huge temporary table that could consume substantial resources if not handled carefully.

#### Performance Considerations
Efficient memory and computational resource utilization are crucial.

#### First-Principles Solution
Generate Date Sequence:
Use a generated date sequence within the data warehouse or an external system like Python/Java to generate a sequence of dates between start_date and end_date.
Dynamic SQL or CTEs:
Utilize Common Table Expressions (CTEs) or Window Functions to join the subscription table with the generated date sequence.
Optimization Techniques:
Use Indexes on relevant columns (subscription_id, start_date, end_date).

#### Cons
ider partitioning the tables for faster query performance.

#### Implementation
Generate Date Sequence (Using a Generated Table):
SQL
WITH RECURSIVE date_sequence AS (
    SELECT MIN(start_date) as start, MAX(end_date) as end
    FROM subscriptions

    UNION ALL

    SELECT DATEADD(day, 1, start), end
    FROM date_sequence
    WHERE start < end
)
SELECT * FROM date_sequence;
Join with Subscriptions Table:
SQL
WITH RECURSIVE date_sequence AS (
    SELECT MIN(start_date) as start, MAX(end_date) as end
    FROM subscriptions

    UNION ALL

    SELECT DATEADD(day, 1, start), end
    FROM date_sequence
    WHERE start < end
),
daily_active_rows AS (
    SELECT s.subscription_id, d.date_day
    FROM subscriptions s
    JOIN date_sequence d ON s.start_date <= d.date_day AND s.end_date >= d.date_day
)
SELECT * FROM daily_active_rows;
Optimizations:
Ensure that the subscriptions table has indexes on start_date and end_date.
Partition the date_sequence CTE to reduce computation.

#### Closing Summary
+ Common Mistakes/Follow-Ups
The above solution leverages recursive common table expressions (CTEs) to dynamically generate date sequences, avoiding materializing a huge cross product.
Important optimizations include indexing, partitioning, and ensuring efficient join conditions.
Potential follow-ups could involve discussing how this approach scales with larger datasets or exploring other database-specific features like window functions or lateral joins.

**Comparison:**

               | 1             | 2023-01-01   | 2023-01-05   |
               | 2             | 2023-01-03   | 2023-01-06   |
               | 2023-01-01    | 2023-01-05   |
               | 2023-01-02    | 2023-01-06   |
                   | 1              | 2023-01-01   |
                   | 1              | 2023-01-02   |
                   | 1              | 2023-01-03   |
                   | 1              | 2023-01-04   |
                   | 1              | 2023-01-05   |
                   | 2              | 2023-01-03   |
                   | 2              | 2023-01-04   |
                   | 2              | 2023-01-05   |
                   | 2              | 2023-01-06   |

**Code:**

```sql
WITH RECURSIVE date_sequence AS (
    SELECT MIN(start_date) as start, MAX(end_date) as end
    FROM subscriptions

    UNION ALL

    SELECT DATEADD(day, 1, start), end
    FROM date_sequence
    WHERE start < end
),
daily_active_rows AS (
    SELECT s.subscription_id, d.date_day
    FROM subscriptions s
    JOIN date_sequence d ON s.start_date <= d.date_day AND s.end_date >= d.date_day
)
SELECT * FROM daily_active_rows;
```

**Example input / output:**

```text
Input:
subscriptions = [
    (1, '2023-01-01', '2023-01-05'),
    (2, '2023-01-03', '2023-01-06')
]

Output:
daily_active_rows = [
    (1, '2023-01-01'),
    (1, '2023-01-02'),
    (1, '2023-01-03'),
    (1, '2023-01-04'),
    (1, '2023-01-05'),
    (2, '2023-01-03'),
    (2, '2023-01-04'),
    (2, '2023-01-05'),
    (2, '2023-01-06')
]
```

**Data:**

```sql
subscriptions: (subscription_id INT, start_date DATE, end_date DATE)

date_sequence: (start DATE, end DATE)
daily_active_rows: (subscription_id INT, date_day DATE)
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Generate Date Sequence
7. Is start < end?
8. Recursively Generate Dates
9. Add Day to Start
10. Check if Date <= end
11. Return Date
12. End Recursion
13. End Sequence Generation
14. Join Subscriptions with DateSequence
15. Filter Rows by Date Range
16. Output Active Daily Rows

