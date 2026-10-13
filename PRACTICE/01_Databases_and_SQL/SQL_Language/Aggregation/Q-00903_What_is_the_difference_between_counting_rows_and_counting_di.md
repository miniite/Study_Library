---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between counting rows and counting distinct groups produced by GROUP BY?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between counting rows and counting distinct groups produced by GROUP BY, we need to delve into the core concepts of SQL aggregate functions and how they operate on data. Let's break this down step-by-step from first principles.

#### Triaging the Question

The question is essentially asking us to compare two common operations in SQL: simple row count (using COUNT(*)) versus counting distinct values within groups (using GROUP BY combined with COUNT(DISTINCT)). The interviewer might be probing your understanding of how these functions work under the hood, their differences, and use cases.

Step-by-Step Solution
1. Counting Rows (COUNT(*))

The COUNT(*) function returns the number of rows in a query result set. It counts every row that is returned by the query, including duplicates and NULLs if they are selected in the query.

#### Example

SQL
SELECT COUNT(*)
FROM orders;

This will return the total number of rows in the orders table, counting all records regardless of their values.

2. Counting Distinct Groups (GROUP BY + COUNT(DISTINCT))

The GROUP BY clause is used to break down data into groups based on one or more columns. When combined with COUNT(DISTINCT), it counts the number of unique occurrences within each group, effectively removing duplicates.

#### Example

SQL
SELECT product_id, COUNT(DISTINCT customer_id)
FROM orders
GROUP BY product_id;

This query will return a result set where each row represents a distinct product_id and the count of different customers who have ordered that product.

Key Differences
Row Count (COUNT(*)): This function simply counts all rows returned by the query, including duplicates.
Distinct Group Count (COUNT(DISTINCT) + GROUP BY): This function first groups data using GROUP BY, then counts unique values within each group. It effectively removes duplicate entries in the count.

#### Closing Summary
and Common Mistakes

In summary, counting rows with COUNT(*) gives you an aggregate number of all records without any filtering or grouping, whereas COUNT(DISTINCT) + GROUP BY allows you to calculate distinct counts within specific groups defined by your query. Misunderstanding these concepts can lead to incorrect assumptions about data distribution and analysis results.

#### Common Mistakes
Misinterpreting the Need for Distinct: Sometimes developers might use COUNT(*) where they should be using COUNT(DISTINCT) if there's a need to avoid counting duplicates.
Ignoring GROUP BY: Failing to understand that GROUP BY is necessary before applying aggregate functions like COUNT(DISTINCT) can lead to incorrect results or missing essential data.

#### Example
Input-Output

Let's consider an example with sample input and expected output:

Sample Data
SQL
CREATE TABLE orders (
    order_id INT,
    product_id INT,
    customer_id INT
);

INSERT INTO orders (order_id, product_id, customer_id) VALUES
(1, 101, 200),
(2, 101, 200),
(3, 102, 201),
(4, 102, 202);
Expected Output

For the query:

SQL
SELECT product_id, COUNT(DISTINCT customer_id)
FROM orders
GROUP BY product_id;

The expected output would be:

PLAINTEXT
+------------+----------------+
| product_id | COUNT(DISTINCT)|
+------------+----------------+
|      101   |         1      |
|      102   |         2      |
+------------+----------------+

This shows that for product_id 101, there is only one distinct customer (200), and for product_id 102, there are two distinct customers (201 and 202).

Self-Check

The solution covers the core concepts of aggregate functions, provides clear examples, highlights common mistakes, and ensures completeness.

**Comparison:**

| order_id | product_id | customer_id |
|----------|------------|-------------|
|     1    |       101  |        200  |
|     2    |       101  |        200  |
|     3    |       102  |        201  |
|     4    |       102  |        202  |

**Code:**

```sql
-- Sample data creation and insertion
CREATE TABLE orders (
    order_id INT,
    product_id INT,
    customer_id INT
);

INSERT INTO orders (order_id, product_id, customer_id) VALUES
(1, 101, 200),
(2, 101, 200),
(3, 102, 201),
(4, 102, 202);
```

**Example input / output:**

```text
+------------+----------------+
| product_id | COUNT(DISTINCT)|
+------------+----------------+
|      101   |         1      |
|      102   |         2      |
+------------+----------------+
```

**Data:**

```plaintext
CREATE TABLE orders (
    order_id INT,
    product_id INT,
    customer_id INT
);

INSERT INTO orders (order_id, product_id, customer_id) VALUES
(1, 101, 200),
(2, 101, 200),
(3, 102, 201),
(4, 102, 202);
```

**Flow:**

1. Row Count
2. Distinct Groups
3. Start
4. Is the question about rowcount or distinct groups?
5. Use COUNT(*)
6. GROUP BY +COUNT(DISTINCT)
7. Count all rows
8. Group by column(s) thencount unique values
9. Output total rows
10. Output distinct counts pergroup
11. End

