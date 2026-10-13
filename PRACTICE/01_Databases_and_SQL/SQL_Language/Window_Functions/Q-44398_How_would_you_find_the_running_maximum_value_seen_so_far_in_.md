---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the running maximum value seen so far in an ordered set using MAX() OVER?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the running maximum value seen so far in an ordered set, we can use window functions in SQL. Specifically, MAX() combined with the OVER clause allows us to calculate a cumulative maximum across the rows of our result set based on some order.

#### Triage

The question is asking how to compute the running maximum (i.e., the highest value encountered so far) within an ordered dataset.
We are using window functions, which provide a way to aggregate data over a range of rows relative to each row in the result set.
Our key assumption is that the input data has some sort of ordering or sequence, such as time-stamped events.

Step-by-step solution from first principles:

Identify the column(s) on which you need to order the dataset. This could be based on a timestamp, an ID, or any other unique identifier.
Use the MAX() function with the OVER clause to define a window that includes all rows up to and including the current row in the sorted sequence.

Here’s how it would look:

SQL
SELECT
    id,
    value,
    MAX(value) OVER (ORDER BY id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_max
FROM your_table_name;

Explanation:

id and value are placeholders for the actual columns in your table. Replace them with the relevant column names.
The MAX() function is used to find the maximum value.
The OVER (ORDER BY id ...) clause defines the window over which we want to calculate the maximum, starting from the first row up to the current row (ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW).
The result will include an additional column called running_max, which contains the running maximum value seen so far for each row.

#### Closing summary
The use of window functions like MAX() OVER is a powerful way to perform calculations over a set of rows that are related to the current row, but not necessarily in the same partition. It allows us to maintain context across multiple rows while performing aggregations.

#### Common mistakes
and follow-ups:

Forgetting to order the data properly (e.g., using ORDER BY).
Misunderstanding how the ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW clause works.
Not considering edge cases, such as what happens if there are duplicate maximum values in your dataset.

**Code:**

```sql
SELECT
    id,
    value,
    MAX(value) OVER (ORDER BY id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_max
FROM your_table_name;
```

