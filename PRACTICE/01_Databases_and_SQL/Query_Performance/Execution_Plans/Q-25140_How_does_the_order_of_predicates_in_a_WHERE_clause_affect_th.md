---
type: "practice"
difficulty: "Intermediate"
---

# How does the order of predicates in a WHERE clause affect the plan and does the optimizer care?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, the order of predicates in a WHERE clause can significantly impact the execution plan chosen by the query optimizer. The optimizer aims to choose the most efficient plan based on various factors including available indexes, table statistics, and cost estimation methods. Understanding how predicate ordering affects the plan helps in optimizing queries for better performance.

The key idea here is that different orders of predicates might lead the optimizer to use a more or less effective execution strategy. For example, if one predicate can quickly filter out many rows, placing it first could reduce the number of subsequent operations needed. This triage step ensures we cover all potential optimization paths.

To illustrate, consider a WHERE clause with multiple conditions: WHERE age > 25 AND city = 'New York' AND gender = 'M'. The optimizer will evaluate different orders of these predicates to determine which plan is most cost-effective, often using statistics and an estimation of the selectivity of each predicate.

Let’s break this down step-by-step:

Understanding Cost-Based Optimization:

The query optimizer uses a cost model to estimate the cost (time or resources) of different execution plans.
This cost model is influenced by factors like index access, table scans, and join operations.

Predicate Evaluation Order:

When predicates are evaluated first, they can quickly reduce the number of rows processed in subsequent steps.
For example, if city = 'New York' significantly filters down the dataset before age > 25, it might be more efficient to place that condition earlier.

#### Example
with a Simple Table:

#### Cons
ider a table employees with columns: id, name, age, city.
If we have an index on city and another one on age.

Predicate Order Analysis:

Placing city = 'New York' first:
SQL
SELECT * FROM employees WHERE city = 'New York' AND age > 25;
The optimizer might use the index on city to quickly filter out non-matching rows, then apply the age predicate.
Placing age > 25 first:
SQL
SELECT * FROM employees WHERE age > 25 AND city = 'New York';
The optimizer might consider a full table scan initially and only later use the index on city.

Optimizer Preferences:

In many databases, the optimizer tends to place more selective conditions (those with lower cardinality) earlier.
This is because such conditions can prune rows faster, reducing the overall cost.

#### Common Mistakes

Ignoring predicate order and assuming the database will always choose the optimal plan without manual intervention.
Not considering statistics updates or schema changes that might alter the optimizer’s decision-making process.

**Comparison:**

```
Predicate Order	Plan Details
City First	Index scan on city + Filter by age
Age First	Full table scan + Filter by city
```

**Code:**

```sql
-- Example SQL queries with different predicate orders

-- Optimal case: index on city is used first
SELECT * FROM employees WHERE city = 'New York' AND age > 25;

-- Potentially suboptimal case: full table scan initially
SELECT * FROM employees WHERE age > 25 AND city = 'New York';
```

**Example input / output:**

```text
-- Initial plan with city first
EXPLAIN ANALYZE SELECT * FROM employees WHERE city = 'New York' AND age > 25;

-- Initial plan with age first
EXPLAIN ANALYZE SELECT * FROM employees WHERE age > 25 AND city = 'New York';
```

**Data:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT,
    city VARCHAR(100)
);

-- Inserting sample data (for demonstration purposes)
INSERT INTO employees (id, name, age, city) VALUES
(1, 'John', 30, 'New York'),
(2, 'Jane', 25, 'Los Angeles'),
(3, 'Mike', 40, 'New York'),
(4, 'Sara', 35, 'Chicago');
```

**Flow:**

1. Yes
2. No
3. Predicate Order
4. Is index on 'city' moreselective?
5. Use index on 'city' first, filterby 'age'
6. Full table scan, apply all filters

