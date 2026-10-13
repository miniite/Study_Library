---
type: "practice"
difficulty: "Expert"
---

# Describe when the planner chooses a merge join, how adding matching indexes on both sides converted a spilling hash join into a cheap merge join, and when the sort cost isn't worth it.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Merge joins are chosen by the SQL query planner in scenarios where both input relations can be sorted efficiently, and the data is evenly distributed between them. Adding indexes on both sides of the join can transform a spilling hash join into a merge join by allowing the database to sort each relation quickly and cheaply, thus reducing sorting costs. However, if the cost of sorting outweighs the benefits, the planner may choose a different join strategy.

#### Triaging the Question
The question delves into how SQL query planners make decisions about which join strategies to use, particularly focusing on merge joins versus hash joins. It also touches on the impact of adding indexes and considers the trade-offs involved in choosing between sorting operations.

#### Assumptions

We are dealing with a database system that uses a cost-based optimizer (CBO) for query planning.
The input relations have sufficient data to make sorting meaningful.
The planner has accurate statistics about the size of the relations and their distribution.

#### First-Principles Solution

When Merge Joins Are Chosen

Merge joins are typically chosen by the SQL planner when both tables involved in the join can be sorted on the join key(s). This is because merge joins require both input relations to be fully sorted on the join keys, and they can then compare rows one-by-one. The planner makes this decision based on the cost estimates for different join strategies.

Transforming Hash Joins into Merge Joins

Hash joins are generally used when there is a large lookup table (the smaller relation) that needs to be joined with a larger table. However, if indexes can be added to both sides of the join, the planner might convert a hash join into a merge join. Here’s how:

Indexing Both Sides: By adding an index on the join key(s) for both tables, we enable efficient sorting and ordering.
Sorting Efficiency: With sorted data, the merge join can proceed without the need to sort one of the relations separately (a costly operation in hash joins).
Reduced Sorting Cost: If the planner estimates that sorting both sides is cheaper than spilling a hash join, it will opt for merge join.
When the Sort Cost Isn't Worth It

The planner evaluates several factors before deciding on a join strategy:

Data Size and Distribution: Smaller or more evenly distributed relations might be better suited for merge joins.
Index Availability: If indexes are available that can sort the data efficiently, the cost of sorting is reduced.
Join Key Selectivity: More selective join keys (keys with fewer distinct values) make hash joins less efficient and favor merge joins.
Cost Estimates: The planner calculates the estimated cost of each strategy based on factors like CPU usage, memory requirements, and disk I/O.

#### Example Scenario

#### Cons
ider a query joining two tables orders and customers. If there are indexes on both customer_id in orders and id in customers, the planner might choose to sort both relations and perform a merge join. However, if the orders table is significantly larger than customers and sorting it would be very costly (e.g., due to memory constraints), the planner might prefer a hash join instead.

#### Common Mistakes/Follow-Ups
Ignoring Indexes: Not recognizing that indexes can reduce the cost of sorting.
Overestimating Costs: Assuming that any sort is inherently expensive, without considering data size and distribution.
Lack of Statistics: Relying solely on hardcoded assumptions rather than using actual statistics for decision-making.

In summary, the choice between merge joins and hash joins depends on the specific characteristics of the data, available indexes, and the cost estimates calculated by the planner. Adding indexes can sometimes transform a less optimal join strategy into a more efficient one, but the planner must also consider other factors to make an informed decision.

**Comparison:**

```
orders	customers
id	customer_id
index	on customer_id
Join Type	Cost Estimation Factors
Merge	Smaller and evenly distributed relations, available indexes.
Hash	Larger lookup table, less selective join keys.
```

**Code:**

```sql
-- Example SQL query using merge join
SELECT * FROM orders
JOIN customers ON orders.customer_id = customers.id
```

**Example input / output:**

```text
Input:
- Orders table with indexed customer_id
- Customers table with indexed id

Output:
- Efficient merge join execution plan using both indexes for sorting.
- Reduced sorting and joining costs compared to a hash join.
```

**Data:**

```json
{
  "orders": [
    {"id": 1, "customer_id": 201},
    {"id": 2, "customer_id": 202},
    ...
  ],
  "customers": [
    {"id": 201, "name": "Alice"},
    {"id": 202, "name": "Bob"},
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is data sorted?
5. Merge join
6. Hash join
7. Sort both relations efficiently
8. Calculate cost of sorting
9. Proceed with merge join
10. Compare costs, choose moreefficient strategy

