---
type: "practice"
difficulty: "Intermediate"
---

# What does a RIGHT JOIN do and how is it equivalent to a LEFT JOIN with the tables swapped?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A RIGHT JOIN in PostgreSQL returns all rows from the right table, and the matched rows from the left table. It's functionally equivalent to a LEFT JOIN where the tables are swapped. The key difference lies in which table acts as the "right" one.

Question Breakdown: The question asks you to explain what a RIGHT JOIN does and how it is equivalent to a LEFT JOIN with the tables swapped. Essentially, this involves understanding both types of joins—RIGHT JOIN and LEFT JOIN, and recognizing their relationship in terms of which table serves as the primary source.

#### Assumptions

You are familiar with basic SQL operations.
The concept of a JOIN is understood in the context of relational databases, specifically PostgreSQL.
There is no data loss expected from one type of join to another unless specified otherwise by conditions or constraints.
Explanation:

A RIGHT JOIN (or RIGHT OUTER JOIN) returns all rows from the right table and the matched rows from the left table. This means that if there are any records in the right table without a match in the left table, they will still be included in the result set with nulls in the columns of the left table.

Step-by-Step Explanation:

Understanding RIGHT JOIN:

The RIGHT JOIN operation returns all rows from the right (second) table and includes matched rows from the left (first) table.
If there is no match, the result contains nulls in columns of the left table.

Equivalent Operation with Tables Swapped:

A LEFT JOIN typically returns all rows from the left table and matches them with rows from the right table.
By swapping the tables in a LEFT JOIN, it now acts as a RIGHT JOIN.

#### Example Scenario
Let's consider two tables, Orders (left) and Customers (right).

SQL
-- Original RIGHT JOIN
SELECT * FROM Customers
RIGHT JOIN Orders ON Customers.customer_id = Orders.customer_id;

This query will return all customers, including those who haven't placed any orders, along with the corresponding order details.

Swapping Tables for Equivalent LEFT JOIN:

SQL
-- Swapped LEFT JOIN
SELECT * FROM Orders
LEFT JOIN Customers ON Orders.customer_id = Customers.customer_id;

This LEFT JOIN will now return all orders, including those without a customer record (represented as nulls in the Customers columns).

Result Comparison:

Both queries will yield similar results, but the role of each table changes:
In the RIGHT JOIN, Customers is on the "right" side.
In the swapped LEFT JOIN, Orders is on the "left" side.

#### Closing Summary
and Common Mistakes/Follow-Ups:
The main takeaway is that a RIGHT JOIN can be thought of as a LEFT JOIN with the tables in reverse order. This understanding helps in managing data and ensuring no rows are lost.
A common mistake might involve assuming that both joins return identical results without considering the role of each table.

#### Follow-up
questions could include:
How would you handle NULL values specifically?
What implications does this have on performance or storage requirements?

