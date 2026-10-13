---
type: "practice"
difficulty: "Basic"
---

# What is the difference between UNION and UNION ALL when combining data from two source tables into one training set?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To combine data from two source tables into a single training dataset, UNION and UNION ALL are commonly used in SQL. The key difference between them lies in how they handle duplicate rows.

TL;DR:
UNION: Removes duplicate rows.
UNION ALL: Includes all rows, including duplicates.

#### Triaging the Question

The question is asking you to differentiate between UNION and UNION ALL. The interviewer might be probing your understanding of SQL operations and how they handle data. You should also clarify if there are any assumptions about the source tables or the dataset.

Step-by-Step Solution from First Principles
1. Understanding UNION

UNION is an operation that combines two result sets into one, but it does so by removing duplicate rows based on all columns involved in the union. To ensure this, you need to include ALL at the end of your query to explicitly tell SQL not to remove duplicates.

SQL
SELECT column1 FROM table1
UNION ALL
SELECT column1 FROM table2;
2. Understanding UNION ALL

UNION ALL, on the other hand, combines two result sets into one and includes all rows from both sources, including any duplicate rows. This means that if a row exists in both table1 and table2, it will appear twice in the resulting dataset.

SQL
SELECT column1 FROM table1
UNION ALL
SELECT column1 FROM table2;
Practical Example

Let's consider two tables, orders_table_1 and orders_table_2, with a common customer ID field.

Table: orders_table_1
order_id	customer_id
1	Alice
2	Bob
Table: orders_table_2
order_id	customer_id
3	Charlie
4	Alice
Using UNION
SQL
SELECT * FROM orders_table_1
UNION ALL
SELECT * FROM orders_table_2;

Output:

order_id	customer_id
1	Alice
2	Bob
3	Charlie
4	Alice

Note that "Alice" appears twice, but only once if using UNION.

Using UNION
SQL
SELECT * FROM orders_table_1
UNION
SELECT * FROM orders_table_2;

Output:

order_id	customer_id
1	Alice
2	Bob
3	Charlie

In this case, "Alice" appears only once.

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Using UNION without considering whether duplicates should be removed.

#### Follow-Up
The interviewer might ask about performance differences or when to use each operation. For example:
UNION ALL is faster because it doesn't perform the additional step of checking for and removing duplicates.
Use UNION when you want unique rows in your result set.

#### Conclusion

Understanding the behavior of UNION and UNION ALL is crucial for ensuring that your SQL queries produce the correct results, especially when dealing with data from multiple sources. Always consider whether duplicates should be removed or not based on the requirements of your analysis.

**Code:**

```sql
-- Example using UNION
SELECT * FROM orders_table_1
UNION
SELECT * FROM orders_table_2;

-- Example using UNION ALL
SELECT * FROM orders_table_1
UNION ALL
SELECT * FROM orders_table_2;
```

**Example input / output:**

```text
-- Example using UNION
SELECT * FROM orders_table_1
UNION
SELECT * FROM orders_table_2;

-- Output:
+----------+-------------+
| order_id | customer_id |
+----------+-------------+
| 1        | Alice       |
| 2        | Bob         |
| 3        | Charlie     |

-- Example using UNION ALL
SELECT * FROM orders_table_1
UNION ALL
SELECT * FROM orders_table_2;

-- Output:
+----------+-------------+
| order_id | customer_id |
+----------+-------------+
| 1        | Alice       |
| 2        | Bob         |
| 3        | Charlie     |
| 4        | Alice       |
```

**Data:**

```
MARKDOWN
Copy
| order_id | customer_id |
|----------|-------------|
| 1        | Alice       |
| 2        | Bob         |
| 3        | Charlie     |
| 4        | Alice       |

| order_id | customer_id |
|----------|-------------|
| 1        | Alice       |
| 2        | Bob         |
| 3        | Charlie     |
```

