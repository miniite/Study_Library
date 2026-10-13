---
type: "practice"
difficulty: "Intermediate"
---

# How do you find customers who have never placed an order using a JOIN?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find customers who have never placed an order, we need to identify those customers whose IDs are not present in the orders table. This problem can be solved using a LEFT JOIN or a subquery approach. Let's explore both methods.

#### Triaging the Question
What is being asked? The question asks for a query that identifies customers who have never placed an order.
Understanding the Data Schema:
We need at least two tables: customers and orders.
The customers table contains customer details (e.g., customer_id, name).
The orders table contains order details with a foreign key to the customer_id.

#### First-Principles Solution
Method 1: Using LEFT JOIN

A LEFT JOIN between the customers and orders tables will return all customers, along with their orders. Customers who have no corresponding rows in the orders table (i.e., those without any orders) will show up as NULL values for the order-related columns.

Method 2: Using Subquery

A subquery can be used to filter out customer IDs that do not appear in the orders table, and then select customers based on these IDs.

Detailed Solution:

Let's start with the LEFT JOIN approach:

SQL
SELECT
    c.customer_id,
    c.name
FROM
    customers c
LEFT JOIN
    orders o ON c.customer_id = o.customer_id
WHERE
    o.order_id IS NULL;

This query returns all customers from the customers table, and only includes those who do not have any corresponding order in the orders table. The WHERE clause filters out rows where there is no match in the orders table.

Alternatively, we can use a subquery:

SQL
SELECT
    customer_id,
    name
FROM
    customers
WHERE
    customer_id NOT IN (
        SELECT
            customer_id
        FROM
            orders
    );

This query selects customers whose IDs are not present in the orders table. The subquery retrieves all customer_ids from the orders table, and then the outer query filters out those IDs.

#### Closing Summary

Both methods yield the same result: a list of customer IDs and names who have never placed an order. The LEFT JOIN approach is generally more efficient as it processes fewer rows before filtering, whereas the subquery might be simpler to understand for beginners.

#### Common Mistakes/Follow-Ups
Using INNER JOIN Instead: This will only return customers with orders.
Ignoring NULL Values: Ensuring you handle NULL values correctly in your query.

#### Performance Considerations
Depending on the size of the tables, a LEFT JOIN might be more performant than a subquery.

Would you like to explore how this solution scales or consider any additional constraints? For instance, we could add conditions for specific dates or types of orders if needed.

**Comparison:**

```
table {
  title "Customers Table";
  row1 "customer_id", "name"
  row2 "1", "Alice"
  row3 "2", "Bob"
  row4 "3", "Charlie"
}

table {
  title "Orders Table";
  row1 "order_id", "customer_id"
  row2 "101", "1"
  row3 "102", "2"
}
```

**Code:**

```sql
-- LEFT JOIN approach
SELECT
    c.customer_id,
    c.name
FROM
    customers c
LEFT JOIN
    orders o ON c.customer_id = o.customer_id
WHERE
    o.order_id IS NULL;

-- Subquery approach
SELECT
    customer_id,
    name
FROM
    customers
WHERE
    customer_id NOT IN (
        SELECT
            customer_id
        FROM
            orders
    );
```

**Example input / output:**

```text
LEFT JOIN approach:
Input:
SELECT
    c.customer_id,
    c.name
FROM
    customers c
LEFT JOIN
    orders o ON c.customer_id = o.customer_id
WHERE
    o.order_id IS NULL;

Output:
customer_id | name
-------------|------
2           | Bob
3           | Charlie

Subquery approach:
Input:
SELECT
    customer_id,
    name
FROM
    customers
WHERE
    customer_id NOT IN (
        SELECT
            customer_id
        FROM
            orders
    );

Output:
customer_id | name
-------------|------
2           | Bob
3           | Charlie
```

**Data:**

```json
{
  "customers": [
    { "customer_id": 1, "name": "Alice" },
    { "customer_id": 2, "name": "Bob" },
    { "customer_id": 3, "name": "Charlie" }
  ],
  "orders": [
    { "order_id": 101, "customer_id": 1 },
    { "order_id": 102, "customer_id": 2 }
  ]
}
```

**Flow:**

1. Subquery
2. LEFT JOIN
3. LEFT JOIN
4. Subquery
5. Start
6. Are you using a LEFT JOIN orsubquery?
7. leftjoin
8. subquery
9. Query the customers who haveno corresponding order
10. SELECT c.customer_id, c.nameFROM customers c LEFT JOINorders o ON c.customer_id =o.customer_id WHEREo.order_id IS NULL
11. SELECT customer_id, nameFROM customers WHEREcustomer_id NOT IN (SELECTcustomer_id FROM orders)

