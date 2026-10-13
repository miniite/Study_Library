---
type: "practice"
difficulty: "Intermediate"
---

# Can you filter on two different aggregate conditions in the same HAVING clause? Give an example.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ GROUP BY and HAVING](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this intermediate-level question, we're exploring whether it's possible to apply multiple aggregate functions within a single HAVING clause and how such queries can be structured in SQL. The key idea is understanding the syntax and logic behind using aggregate functions with conditions in the HAVING clause.

#### Triage

The question asks if it's feasible to use two different aggregate conditions simultaneously within a HAVING clause. This involves understanding how HAVING works, which filters groups after aggregates are calculated, as opposed to WHERE, which filters rows before aggregation.

#### First-Principles Solution

In SQL, the HAVING clause allows us to apply conditions on aggregated values. We can indeed use multiple aggregate functions within a single HAVING clause by combining them with logical operators like AND, OR, or even comparison operations directly involving those aggregates.

#### Example Scenario

Suppose we have an orders table with the following schema:

SQL
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10,2)
);

We want to find customers who have made at least 5 orders and each of those orders has an average amount greater than $100.

Step-by-Step Solution

Here’s how we can write this query:

SQL
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) >= 5 AND AVG(amount) > 100;
Explanation:
GROUP BY: We group the data by customer_id to aggregate per customer.
HAVING: The HAVING clause applies conditions after the aggregation is performed. Here, we use two conditions combined with an AND operator.
COUNT(order_id) >= 5: Ensures that each group has at least 5 orders.
AVG(amount) > 100: Ensures that the average amount of those orders is more than $100.

#### Closing Summary

Yes, it is possible to filter on two different aggregate conditions in a single HAVING clause. The key is understanding how GROUP BY and HAVING work together to first perform aggregation and then apply filtering based on the results.

#### Common Mistakes/Follow-Ups
Mistake: Using WHERE instead of HAVING. WHERE filters rows before grouping, so it won't be able to reference aggregate functions directly.

#### Follow-Up
What happens if one condition is not met? The entire group will still be included in the results because both conditions must hold true for a group.

#### Example
Output

#### Cons
ider the following sample data:

SQL
INSERT INTO orders (order_id, customer_id, amount) VALUES
(101, 1, 200.5),
(102, 1, 300.0),
(103, 1, 400.0),
(104, 1, 500.0),
(105, 1, 600.0),
(106, 2, 700.0),
(107, 2, 800.0),
(108, 3, 900.0),
(109, 3, 1000.0);

Running the query:

SQL
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) >= 5 AND AVG(amount) > 100;

Would return customer_id as there are no customers with at least 5 orders and an average amount greater than $100.

**Comparison:**

| order_id | customer_id | amount   |
|----------|-------------|----------|
| 101      | 1           | 200.5    |
| 102      | 1           | 300.0    |
| 103      | 1           | 400.0    |
| 104      | 1           | 500.0    |
| 105      | 1           | 600.0    |
| 106      | 2           | 700.0    |
| 107      | 2           | 800.0    |
| 108      | 3           | 900.0    |
| 109      | 3           | 1000.0   |

**Code:**

```sql
-- Sample data
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10,2)
);

INSERT INTO orders (order_id, customer_id, amount) VALUES
(101, 1, 200.5),
(102, 1, 300.0),
(103, 1, 400.0),
(104, 1, 500.0),
(105, 1, 600.0),
(106, 2, 700.0),
(107, 2, 800.0),
(108, 3, 900.0),
(109, 3, 1000.0);

-- Query
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) >= 5 AND AVG(amount) > 100;
```

**Example input / output:**

```text
-- Sample Input
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) >= 5 AND AVG(amount) > 100;

-- Expected Output (none)
```

**Data:**

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10,2)
);

INSERT INTO orders (order_id, customer_id, amount) VALUES
(101, 1, 200.5),
(102, 1, 300.0),
(103, 1, 400.0),
(104, 1, 500.0),
(105, 1, 600.0),
(106, 2, 700.0),
(107, 2, 800.0),
(108, 3, 900.0),
(109, 3, 1000.0);
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Does customer have >=5orders?
7. Calculate average amount
8. Is the average amount > $100?
9. Customer meets conditions
10. Customer does not meetconditions

