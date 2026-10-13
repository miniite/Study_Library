---
type: "practice"
difficulty: "Intermediate"
---

# How would you read a simple EXPLAIN output to tell if an index is being used?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN ANALYZE](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether an index is being used in a query, one of the first steps is to examine the EXPLAIN output provided by your database system. This output gives you insights into how the database engine plans to execute the query. For this explanation, we'll use PostgreSQL as our example, but the principles apply broadly.

The key lines to look for are those related to index usage. If an index is being used, you’ll see references to that in the EXPLAIN output. Let’s break it down step by step.

**Comparison:**

| QUERY PLAN               | EXPLANATION |
|--------------------------|-------------|
| Index Scan using...       | Uses an index to retrieve data. |
| cost=0.29..8.30          | Estimated cost of the operation.|
| rows=1 width=467         | Expected number of rows returned and estimated memory usage.|

**Code:**

```sql
-- Example SQL query and its EXPLAIN output
EXPLAIN SELECT * FROM employees WHERE department_id = 10;
```

**Example input / output:**

```

The EXPLAIN output will look something like this:

PLAINTEXT
Copy
                                                      QUERY PLAN
-------------------------------------------------------------------------------------------------
 Index Scan using employees_department_idx on employees  (cost=0.29..8.30 rows=1 width=467)
   Index Cond: (department_id = 10)
(2 rows)
```

**Data:**

```plaintext
-- Sample EXPLAIN output for the above query
"Index Scan using employees_department_idx on employees"
```

**Flow:**

1. Yes
2. No
3. Run EXPLAIN on query
4. Does output mention index?
5. Index is being used
6. Index not being used or otheroperations
7. Check index name andconditions for more details
8. Consider alternative indexes orquery optimizations

