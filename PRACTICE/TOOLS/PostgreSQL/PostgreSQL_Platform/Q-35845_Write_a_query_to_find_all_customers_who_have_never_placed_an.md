---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find all customers who have never placed an order using a LEFT JOIN or NOT EXISTS.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify all customers who do not have any corresponding orders in our database. This can be achieved either through a LEFT JOIN or a NOT EXISTS clause. Let's first break down the requirements and assumptions before diving into the solution.

#### Triage

The question asks for a query to find customers with no orders, which translates to finding all rows from one table (customers) that do not have any matching rows in another related table (orders). We will use LEFT JOIN or NOT EXISTS to achieve this. The main assumptions are:

There is a customer table containing customer information.
There is an order table where each order has a reference to the corresponding customer.
Step-by-Step Solution

Using LEFT JOIN:

We will join the customer and order tables on the foreign key (e.g., customer_id) that links customers to orders.
The result of this join will include all customers, even those who have no corresponding orders in the order table.
Any row from the customer table without a match in the order table will be represented by NULL values for columns coming from the order side.

Using NOT EXISTS:

We will filter out customers whose orders exist in the order table.
The NOT EXISTS clause ensures that only rows from the customer table with no corresponding entries in the order table are selected.

#### Example
Queries
Using LEFT JOIN:
SQL
SELECT c.customer_id, c.name, o.order_id
FROM customer AS c
LEFT JOIN order AS o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
Using NOT EXISTS:
SQL
SELECT *
FROM customer AS c
WHERE NOT EXISTS (SELECT 1 FROM order AS o WHERE o.customer_id = c.customer_id);

#### Closing Summary
and Common Mistakes/Follow-Ups

In both approaches, we are filtering out customers who have placed orders. The LEFT JOIN approach explicitly shows which customers do not have any orders with NULL values for the order columns. The NOT EXISTS approach directly excludes such customers.

#### Common mistakes
include:

Forgetting to use a condition in the WHERE clause (e.g., o.order_id IS NULL) when using LEFT JOIN.
Incorrectly placing the NOT EXISTS subquery, which should be correlated with the main query.

#### Follow-ups
might include discussing performance considerations between these two approaches or considering additional filtering criteria if there are more complex business requirements.

