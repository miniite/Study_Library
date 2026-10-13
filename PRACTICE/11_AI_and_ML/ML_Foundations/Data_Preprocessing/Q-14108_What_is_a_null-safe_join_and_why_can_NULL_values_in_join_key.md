---
type: "practice"
difficulty: "Intermediate"
---

# What is a null-safe join and why can NULL values in join keys silently drop rows during a cleaning merge?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, a null-safe join (also known as an outer or left outer join) ensures that all records from one table are retained even if there is no matching record in the other table. In the context of data preprocessing and cleaning, this becomes particularly important because NULL values in join keys can silently drop rows during a merge operation. This answer will cover why this happens, explain what null-safe joins do to prevent it, and provide an example for clarity.

#### Triage

The question is asking you to understand how NULL values in join keys can affect the result of a data merge (join) operation and introduces the concept of using a null-safe join as a solution. The key assumption here is that both tables being joined have columns with potential NULL values, which could lead to rows being dropped during the merge.

#### First-Principles Solution

When performing a standard inner or left join on two tables based on a common key (e.g., customer_id), if either of the customer_id values in one of the records is NULL, that record will be excluded from the resulting dataset. This can lead to data loss, especially when cleaning and preprocessing datasets where such missing values are common.

A null-safe join ensures that even if a value is NULL on one side, it still retains rows as long as there’s a match on the other side. In SQL terms, this means using LEFT JOIN with conditions like ON (t1.customer_id = t2.customer_id OR (t1.customer_id IS NULL AND t2.customer_id IS NULL)).

#### Example

Let's consider two tables: orders and customers. The goal is to match each order with its corresponding customer.

Sample Data
PLAINTEXT
Table: orders
id   |  order_id    |  customer_id
-----|-------------|--------------
1    |  O001        |  C101
2    |  O002        |  NULL
3    |  O003        |  C103

Table: customers
id   |  customer_id |  name
-----|-------------|---------
4    |  C101        |  Alice
5    |  C102        |  Bob
6    |  NULL        |  Carol
Standard Inner Join
SQL
SELECT orders.order_id, customers.name
FROM orders
INNER JOIN customers ON orders.customer_id = customers.customer_id;

This will result in:

PLAINTEXT
order_id | name
---------|------
O001     | Alice

Rows for O002 and O003 are dropped because the join key is NULL on one side.

Null-Safe Join

To avoid dropping these rows, we use a null-safe condition in our JOIN:

SQL
SELECT orders.order_id, customers.name
FROM orders
LEFT JOIN customers ON (orders.customer_id = customers.customer_id OR (orders.customer_id IS NULL AND customers.customer_id IS NULL));

This will result in:

PLAINTEXT
order_id | name
---------|------
O001     | Alice
NULL     | NULL  # No customer for O002, but retained as a row.
O003     | Carol # Match found with the NULL value on both sides.

#### Closing Summary
and Common Mistakes/Follow-Ups

Null-safe joins are essential in data preprocessing to ensure that no relevant rows are accidentally dropped due to NULL values. By understanding this concept, you can design more robust ETL processes that handle missing data appropriately.

#### Common mistakes
include not recognizing the need for null-safe joins when dealing with incomplete datasets or assuming inner joins will suffice for all cases. Always consider edge cases like the ones shown above during your preprocessing and cleaning steps.

Depth + Language Match Intermediate Experience

Understanding how to handle NULL values in join keys is crucial, especially when working with large and complex datasets. For instance, in a real-world scenario, you might have multiple NULL values or even inconsistent data types across tables, which can complicate the join operation. Using null-safe joins can help maintain data integrity without losing potentially valuable records.

Self-Check

The answer covers the core concept, provides an example, and discusses common mistakes, ensuring correctness and completeness.

**Comparison:**

```
table "orders" as o
  column "id"
  column "order_id"
  column "customer_id"

table "customers" as c
  column "id"
  column "customer_id"
  column "name"

join "Null-Safe Join" from orders (o) to customers (c)
  on "o.customer_id = c.customer_id OR (o.customer_id IS NULL AND c.customer_id IS NULL)"
```

**Code:**

```sql
-- Standard Inner Join
SELECT orders.order_id, customers.name
FROM orders
INNER JOIN customers ON orders.customer_id = customers.customer_id;

-- Null-Safe Join
SELECT orders.order_id, customers.name
FROM orders
LEFT JOIN customers ON (orders.customer_id = customers.customer_id OR (orders.customer_id IS NULL AND customers.customer_id IS NULL));
```

**Example input / output:**

```text
Input:
orders =
id   |  order_id    |  customer_id
-----|-------------|--------------
1    |  O001        |  C101
2    |  O002        |  NULL
3    |  O003        |  C103

customers =
id   |  customer_id |  name
-----|-------------|---------
4    |  C101        |  Alice
5    |  C102        |  Bob
6    |  NULL        |  Carol

Inner Join:
order_id | name
---------|------
O001     | Alice

Null-Safe Join:
order_id | name
---------|------
O001     | Alice
NULL     | NULL
O003     | Carol
```

**Data:**

```plaintext
Table: orders
id   |  order_id    |  customer_id
-----|-------------|--------------
1    |  O001        |  C101
2    |  O002        |  NULL
3    |  O003        |  C103

Table: customers
id   |  customer_id |  name
-----|-------------|---------
4    |  C101        |  Alice
5    |  C102        |  Bob
6    |  NULL        |  Carol
```

**Flow:**

1. Yes
2. No
3. Start
4. Define the join keys
5. Are both values NULL?
6. Include the row in the result
7. Compare non-NULL values
8. Include the row if they match,else exclude
9. End

