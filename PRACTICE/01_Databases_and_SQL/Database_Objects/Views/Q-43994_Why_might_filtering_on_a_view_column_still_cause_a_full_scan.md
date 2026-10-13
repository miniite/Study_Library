---
type: "practice"
difficulty: "Deep"
---

# Why might filtering on a view column still cause a full scan of the underlying table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Using Views Effectively](<../../../../01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you filter data using a view, it’s common to think that the database will apply the filter directly to the view, which is essentially just a virtual table created based on an SQL query. However, filtering on a view column can still cause a full scan of the underlying table due to how views are implemented and optimized by the database system.

#### Triage

The question asks why filtering on a view might not result in efficient execution, specifically leading to a full table scan even when you expect a subset of data. The key points to consider are:

How Views Are Implemented: A view is essentially an SQL statement stored as a virtual table.
Optimization Decisions by the Database Engine: The database engine makes decisions based on cost-based optimization, which might not always be straightforward when dealing with views.
Core Idea

The fundamental concept here is understanding how the database engine processes queries involving views and why it might choose suboptimal execution plans.

Step-by-Step Solution

Understanding Views:

A view is a virtual table that is created using an SQL query. When you filter on a column of a view, the query optimizer must translate this into a corresponding operation on the underlying tables.

Query Execution Plan:

The database engine needs to generate an execution plan for your query involving the view. This involves analyzing the structure and data distribution in both the view and the underlying table(s).
If the database does not have enough information or cannot make a confident decision about the filter's efficiency, it might opt for a full scan of the base tables.

Filter Pushdown:

In some cases, the database can push down filters directly to the underlying tables, but this is not always possible. Filters on views may require additional operations like joins or subqueries, which could lead to less efficient plans.
The decision to use a full scan depends on factors such as statistics, indexing, and the complexity of the view definition.

Cost-Based Optimization:

Database engines use cost-based optimization to decide the most efficient way to execute queries. This involves estimating the cost (time, disk I/O) of different execution plans.
If the optimizer estimates that a full scan is cheaper than other operations required by the view’s query, it might choose this plan.

#### Example
s and Edge Cases:

#### Cons
ider a view defined with multiple joins or subqueries. The database may not be able to push down filters effectively in such cases.
Views without indexes on critical columns can also lead to full scans when filtering is applied.

#### Closing Summary

In summary, filtering on a view column might still cause a full scan of the underlying table because the query optimizer cannot always efficiently translate the filter through the view. Factors like the complexity of the view definition, lack of appropriate statistics or indexes, and cost-based optimization decisions influence this behavior.

#### Common Mistakes/Follow-Ups
Assuming that filtering on a view will automatically apply to the underlying tables.
Not considering the specific structure and complexity of the view.
Ignoring the role of database statistics and indexing in query optimization.

**Comparison:**

| Product ID | Total Sales |
|------------|-------------|
| 1          | 500         |
| 2          | 700         |
| 3          | 900         |

**Code:**

```

-- Example View Definition CREATE VIEW sales_view AS SELECT product_id, SUM(sales_amount) as total_sales FROM sales_data GROUP BY product_id;

CODE
Copy

```sql
-- Filtering on a view column that involves multiple tables or subqueries
SELECT * FROM sales_view WHERE total_sales > 100000;
```

**Example input / output:**

```text
-- Worked Input -> Output Example
Input:
SELECT * FROM sales_view WHERE total_sales > 100000;

Output (Expected):
No rows returned since the view aggregates data by product_id and no single product has a total_sales greater than 100000.
```

**Data:**

```json
{
  "sales_data": [
    {"product_id": 1, "sales_amount": 500},
    {"product_id": 2, "sales_amount": 700},
    {"product_id": 3, "sales_amount": 900}
  ]
}
```

**Flow:**

1. Yes
2. No
3. View Definition
4. Is Filter Pushable?
5. Apply Filter
6. Full Table Scan
7. Return Results
8. Execute Plan with Full Scan

