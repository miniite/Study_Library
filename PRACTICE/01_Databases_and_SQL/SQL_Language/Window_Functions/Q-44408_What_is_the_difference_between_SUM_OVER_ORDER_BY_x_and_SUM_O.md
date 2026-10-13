---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between SUM() OVER (ORDER BY x) and SUM() OVER (PARTITION BY x)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between SUM() over an ORDER BY clause and a SUM() over a PARTITION BY clause, we need to break down what each does in SQL. Essentially, window functions allow us to perform calculations across a set of table rows that are related to the current row. The key differences lie in how they handle data ordering and partitioning.

Firstly, let's restate the core idea: SUM() over an ORDER BY clause will order all rows before calculating the cumulative sum up to each row. On the other hand, SUM() over a PARTITION BY clause divides the data into partitions based on one or more columns and then performs the aggregation within those partitions.

Now let's dive into a step-by-step first-principles solution:

#### Triage
The question is probing your understanding of window functions, specifically ORDER BY and PARTITION BY.
We are assuming that you have basic knowledge of SQL and aggregate functions.
The input data could be any table with columns relevant to the SUM() function and a column for ordering or partitioning.

#### First-Principles Solution

Understanding ORDER BY:

When using SUM() over an ORDER BY clause, the rows are first ordered based on the specified column(s).
Then, the SUM() function is applied cumulatively across these ordered rows.

#### Example
SQL
SELECT id, value,
       SUM(value) OVER (ORDER BY id) AS cumulative_sum
FROM data_table;
Here, the row with the smallest id will have a cumulative_sum of its own value. The next row’s cumulative_sum would be the sum of its own value and the previous one's cumulative_sum.

Understanding PARTITION BY:

When using SUM() over a PARTITION BY clause, the rows are first grouped into partitions based on the specified column(s).
The SUM() function is then applied within each partition.

#### Example
SQL
SELECT id, value,
       SUM(value) OVER (PARTITION BY group_column) AS sum_within_group
FROM data_table;
In this case, all rows with the same group_column value will be grouped together, and their value will be summed up within that partition.

#### Closing Summary
+ Common Mistakes/Follow-Ups
The main difference is in how the rows are handled: ORDER BY orders the entire dataset before applying the function, while PARTITION BY groups the data first.
A common mistake is assuming both functions behave the same way because they contain a similar syntax. Remember to consider the context of your columns and what you intend to achieve with your query.

#### Example

Let's take an example table:

MARKDOWN
| id | value |
|----|-------|
| 1  | 10    |
| 2  | 20    |
| 3  | 30    |
| 4  | 40    |

Using SUM() over ORDER BY:

SQL
SELECT id, value,
       SUM(value) OVER (ORDER BY id) AS cumulative_sum
FROM data_table;

Output:

MARKDOWN
| id | value | cumulative_sum |
|----|-------|----------------|
| 1  | 10    | 10             |
| 2  | 20    | 30             |
| 3  | 30    | 60             |
| 4  | 40    | 100            |

Using SUM() over PARTITION BY:

SQL
SELECT id, value,
       SUM(value) OVER (PARTITION BY group_column) AS sum_within_group
FROM data_table;

Assuming group_column has values [A, A, B, B], the output would be:

MARKDOWN
| id | value | sum_within_group |
|----|-------|------------------|
| 1  | 10    | 20               |
| 2  | 20    | 20               |
| 3  | 30    | 70               |
| 4  | 40    | 70               |
Self-Check

I have confirmed the correctness and completeness of the solution.

**Comparison:**

| id | value |
|----|-------|
| 1  | 10    |
| 2  | 20    |
| 3  | 30    |
| 4  | 40    |

**Code:**

```sql
-- Example code for ORDER BY
SELECT id, value,
       SUM(value) OVER (ORDER BY id) AS cumulative_sum
FROM data_table;

-- Example code for PARTITION BY
SELECT id, value,
       SUM(value) OVER (PARTITION BY group_column) AS sum_within_group
FROM data_table;
```

**Example input / output:**

| id | value | cumulative_sum |
|----|-------|----------------|
| 1  | 10    | 10             |
| 2  | 20    | 30             |
| 3  | 30    | 60             |
| 4  | 40    | 100            |
| id | value | sum_within_group |
|----|-------|------------------|
| 1  | 10    | 20               |
| 2  | 20    | 20               |
| 3  | 30    | 70               |
| 4  | 40    | 70               |

**Data:**

```
MARKDOWN
Copy
| id | value |
|----|-------|
| 1  | 10    |
| 2  | 20    |
| 3  | 30    |
| 4  | 40    |
```

**Flow:**

1. SUM() OVER ORDER BY
2. Order Rows
3. Apply SUM()
4. Result
5. SUM() OVER PARTITION BY
6. Partition Rows
7. Apply SUM()
8. Result

