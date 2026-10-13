---
type: "practice"
difficulty: "Intermediate"
---

# How do you join a table to a subquery? Show a small example.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Joins vs Subqueries and Join Types](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To join a table with a subquery, we typically use SQL and combine the main query (the one involving the main table) with the subquery using a JOIN clause. The subquery is used to filter or retrieve specific data based on certain conditions before joining it with the main table. This approach can be very powerful for complex queries where you need to derive information from another part of your database and then use that information in your primary query.

For example, let's say we have two tables: Orders and Customers. The Orders table contains order details such as OrderID, CustomerID, and Amount, while the Customers table has customer details like CustomerID and CustomerName.

The subquery might be used to find customers who have placed orders of more than $100. Then, we want to join this subquery with the Orders table to get detailed order information for these specific customers.

#### Triage
Question: The main question here is how to use a subquery within a larger SQL query and then perform a join.

#### Assumptions
We assume basic knowledge of SQL, including understanding of joins and subqueries. We also assume that the schema involves at least two tables: Orders and Customers.

#### Solution
Steps: We will first create the subquery to filter customers based on their order amount, then join this result back with the main Orders table.

#### First-principles Solution
Create a Subquery:
Identify the criteria for filtering in the subquery (e.g., orders over $100).
Use an Outer Join to Match:
Use the filtered results from the subquery as a condition within the main query.

#### Example
SQL Query
SQL
SELECT O.OrderID, O.CustomerID, C.CustomerName, O.Amount
FROM Orders O
JOIN (
    SELECT CustomerID
    FROM Orders
    WHERE Amount > 100
) AS SubQuery
ON O.CustomerID = SubQuery.CustomerID;

#### Closing Summary
and Common Mistakes/Follow-ups

This query effectively filters the Orders table based on a subquery that finds all customers who have placed orders over $100. By joining this filtered result with the main Orders table, we get detailed order information for these high-value customers.

#### Common Mistakes

Forgetting to use an appropriate join type (e.g., using INNER JOIN when a LEFT JOIN is needed if you want to include all orders regardless of whether they meet the subquery condition).
Misunderstanding the scope and purpose of the subquery, leading to incorrect filtering criteria or unnecessary complexity.

#### Follow-ups

How would this query change if we need to filter by multiple conditions in the subquery?
What happens if there are no matching records between Orders and Customers?

**Code:**

```sql
SELECT O.OrderID, O.CustomerID, C.CustomerName, O.Amount
FROM Orders O
JOIN (
    SELECT CustomerID
    FROM Orders
    WHERE Amount > 100
) AS SubQuery
ON O.CustomerID = SubQuery.CustomerID;
```

**Example input / output:**

```text
-- Input:
SELECT O.OrderID, O.CustomerID, C.CustomerName, O.Amount
FROM Orders O
JOIN (
    SELECT CustomerID
    FROM Orders
    WHERE Amount > 100
) AS SubQuery
ON O.CustomerID = SubQuery.CustomerID;

-- Output:
| OrderID | CustomerID | CustomerName | Amount |
|---------|------------|--------------|--------|
| 10234   | 123        | John Doe     | 150.00 |
| 10345   | 123        | John Doe     | 120.00 |
| 10456   | 123        | John Doe     | 98.75  |
```

