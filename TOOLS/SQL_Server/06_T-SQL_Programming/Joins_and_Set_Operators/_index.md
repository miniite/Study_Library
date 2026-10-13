---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Joins_and_Set_Operators"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Joins Subqueries and Set Operators

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Joins combine tables *horizontally*: they match rows from two inputs on a predicate and put their columns side by side. Set operators (UNION, UNION ALL, INTERSECT, EXCEPT) combine query results *vertically*: they stack or compare whole rows. Each has precise NULL, duplicate and ordering rules.

### 1. The big picture
In logical query processing, the `FROM` clause is phase 1. Joins live there. They are table operators that take two input tables and produce one. Set operators sit *between* complete queries: `query1 UNION query2`. Together they are how you combine data in a relational database. Most wrong results in reporting come from misunderstood joins: an outer join cancelled by a `WHERE` filter, duplicates from a one-to-many join, or `NOT IN` against a column with NULLs.

### 2. Core concepts and vocabulary
- **Cross join**: cartesian product. Every row of A paired with every row of B (|A| × |B| rows).
- **Inner join**: cartesian product, then keep pairs where the `ON` predicate is TRUE.
- **Outer join** (`LEFT`, `RIGHT`, `FULL`): inner join, then add back the rows of the **preserved** side(s) that had no match, with NULLs for the other side's columns. These are called **outer rows**.
- **Preserved side**: the table whose rows are all kept. In `A LEFT JOIN B`, A is preserved.
- **Self join**: a table joined to itself under two aliases (employee → manager).
- **Equi-join / non-equi-join**: join on `=` vs. on other operators (`BETWEEN`, `<`).
- **Composite join**: join on several columns (`ON a.k1 = b.k1 AND a.k2 = b.k2`).
- **Semi join**: return rows of A that have a match in B, without B's columns and without duplicating A (`EXISTS`, `IN`).
- **Anti semi join**: rows of A with no match in B (`NOT EXISTS`, `EXCEPT`, careful `NOT IN`).
- **Set operator**: an operator applied to two query results with the same number of columns and compatible types. Columns are matched by position. Result column names come from the first query.
- **UNION / UNION ALL**: all rows from both inputs. `UNION` removes duplicates; `UNION ALL` keeps them.
- **INTERSECT**: distinct rows that appear in both inputs.
- **EXCEPT**: distinct rows of the first input that don't appear in the second. Order of inputs matters.
- **Multiset**: a set that allows duplicates. The standard defines `INTERSECT ALL` / `EXCEPT ALL`; SQL Server does not implement them.

### 3. How it works, step by step
**Logical join processing** (per join, left to right in `FROM`):
1. Cartesian product of the two inputs.
2. Apply the `ON` predicate. Keep only TRUE pairs. NULL = NULL is UNKNOWN, so NULL keys never match.
3. *(Outer joins only)* Add outer rows from the preserved side with NULLs on the other side.

Then `WHERE` runs on the result. This is why **ON and WHERE mean different things for outer joins**:
- A predicate in `ON` decides *matching*. Rows of the preserved side always come back.
- A predicate in `WHERE` is a final *filter*. If it tests a column from the non-preserved side, it removes the outer rows (their values are NULL → UNKNOWN), which silently turns the outer join into an inner join.

**Multi-join queries.** Joins are processed left to right logically; each result becomes the left input of the next. An outer join followed by an inner join on the non-preserved side cancels the outer rows. Fix it by making the second join an outer join too, or by putting the inner join in parentheses: `A LEFT JOIN (B JOIN C ON …) ON …`.

**Set operator rules:**

| Rule | Detail |
|---|---|
| Same number of columns | Matched by position, not by name |
| Compatible types | Lower precedence type converts to higher |
| Names | Taken from the first query |
| ORDER BY | Allowed only once, at the end, on the combined result |
| NULLs | Treated as equal: two NULLs are duplicates (unlike `=`) |
| Precedence | `INTERSECT` binds tighter than `UNION` and `EXCEPT`; use parentheses |

### 4. Worked example
```sql
-- Customers and their orders
SELECT c.custid, c.companyname, o.orderid
FROM Sales.Customers AS c
LEFT JOIN Sales.Orders AS o
  ON o.custid = c.custid;
-- 832 rows: 830 matched orders + 2 customers with no orders (orderid = NULL)

-- Customers with NO orders (anti join via outer join)
SELECT c.custid, c.companyname
FROM Sales.Customers AS c
LEFT JOIN Sales.Orders AS o ON o.custid = c.custid
WHERE o.orderid IS NULL;      -- test a NOT NULL column (PK) of the non-preserved side

-- The classic bug: filter on the non-preserved side in WHERE
SELECT c.custid, o.orderid
FROM Sales.Customers AS c
LEFT JOIN Sales.Orders AS o ON o.custid = c.custid
WHERE o.orderdate >= '20220101';      -- customers with no orders disappear
-- Fix: move the condition into ON
SELECT c.custid, o.orderid
FROM Sales.Customers AS c
LEFT JOIN Sales.Orders AS o
  ON o.custid = c.custid AND o.orderdate >= '20220101';
```

**NOT IN vs. NOT EXISTS with NULLs:**
```sql
-- If any Orders.custid is NULL, this returns ZERO rows:
SELECT custid FROM Sales.Customers
WHERE custid NOT IN (SELECT o.custid FROM Sales.Orders AS o);
-- custid NOT IN (1, 2, NULL)  =  custid<>1 AND custid<>2 AND custid<>NULL → never TRUE

-- Safe:
SELECT c.custid FROM Sales.Customers AS c
WHERE NOT EXISTS (SELECT * FROM Sales.Orders AS o WHERE o.custid = c.custid);
```

**Set operators:**
```sql
-- Locations that have both employees and customers
SELECT country, region, city FROM HR.Employees
INTERSECT
SELECT country, region, city FROM Sales.Customers;

-- Employee locations with no customers; NULL regions compare as equal here
SELECT country, region, city FROM HR.Employees
EXCEPT
SELECT country, region, city FROM Sales.Customers;

-- Combine two years of archive, keep duplicates, sort once at the end
SELECT orderid, orderdate FROM dbo.Orders2024
UNION ALL
SELECT orderid, orderdate FROM dbo.Orders2025
ORDER BY orderdate;
```
Note how `INTERSECT` / `EXCEPT` handle NULL region values for you. Writing the same thing with joins needs `ON e.region = c.region OR (e.region IS NULL AND c.region IS NULL)`.

**Emulating INTERSECT ALL** (keep duplicates up to the smaller count) with `ROW_NUMBER`:
```sql
SELECT country, region, city
FROM (SELECT country, region, city,
             ROW_NUMBER() OVER (PARTITION BY country, region, city ORDER BY (SELECT 0)) AS rn
      FROM HR.Employees
      INTERSECT
      SELECT country, region, city,
             ROW_NUMBER() OVER (PARTITION BY country, region, city ORDER BY (SELECT 0))
      FROM Sales.Customers) AS d;
```

### 5. Under the hood
- **Physical join operators.** The logical join type (inner/outer/semi) is separate from the physical algorithm:
  - **Nested Loops**: for each outer row, seek the inner input. Best when the outer input is small and the inner side has an index on the join key.
  - **Merge Join**: both inputs sorted on the key, walked together. Very cheap if indexes already provide the order; one-to-many is cheap, many-to-many needs a worktable in tempdb.
  - **Hash Match**: build a hash table on the smaller input, probe with the other. Good for large unsorted inputs; needs a memory grant and can spill to tempdb.
  - **Adaptive Join** (batch mode, SQL 2017+): chooses between hash and nested loops at runtime based on the actual row count.
- **Set operators in plans.** `UNION ALL` becomes a *Concatenation* operator: cheap, no sorting. `UNION` adds a duplicate-removal step (Sort Distinct, Hash Aggregate or Merge Join (Union)). `INTERSECT` and `EXCEPT` usually become *Left Semi Join* and *Left Anti Semi Join* plus a distinct step.
- **Anti joins.** `NOT EXISTS` becomes a *Left Anti Semi Join*. `NOT IN` on a nullable column needs extra plan branches to check for NULLs (row count spool, extra scans). Declaring the column `NOT NULL` gives the simpler plan.
- **Join elimination.** With a trusted foreign key and a `NOT NULL` referencing column, the optimizer drops an inner join to the parent table if no parent columns are selected. Views that join many tables benefit from this.
- **Row multiplication.** A join to a "many" side repeats the "one" side rows. `SUM` after such a join double-counts. Aggregate first (in a derived table or APPLY), then join.

### 6. Performance and design implications
- Index foreign key columns. The child side of almost every join is the FK column, and nested loops need a seek there.
- Prefer `UNION ALL` when duplicates are impossible or wanted. `UNION` adds a sort or hash for nothing.
- Prefer `EXISTS` / `NOT EXISTS` for semi and anti joins. They stop at the first match and handle NULLs correctly.
- Join on columns of the same data type. A mismatch (e.g. `varchar` vs. `nvarchar`) causes implicit conversions and can prevent seeks.
- `OR` across different columns in a join predicate often forces scans. Consider splitting into a `UNION ALL` of two joins.
- Measure with `STATISTICS IO` per table and look for Hash Match spills (warning icon) and Sort operators in plans.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INNER / LEFT / RIGHT / FULL [OUTER] JOIN` | Standard joins | Daily |
| `CROSS JOIN` | Cartesian product | Generating combinations, number tables |
| `CROSS APPLY` / `OUTER APPLY` | Join to a table expression that can reference the left row | Top-N per group, calling TVFs |
| `EXISTS` / `NOT EXISTS` | Semi / anti semi join | Existence checks |
| `UNION ALL` | Stack results without duplicate removal | Combining partitions/archives |
| `INTERSECT` / `EXCEPT` | NULL-safe row comparison | Data diff, reconciliation |
| `OPTION (HASH JOIN / LOOP JOIN / MERGE JOIN)` | Forces the join algorithm for the whole query | Testing only, last resort |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `LEFT JOIN … WHERE key IS NULL` | Fine for anti joins, but test a NOT NULL column; `NOT EXISTS` is clearer |
| `NOT EXISTS` | — always safe |
| `NOT IN (subquery)` | Only if the subquery column is NOT NULL |
| `EXCEPT` for comparing two tables | It removes duplicates; use `EXCEPT` both ways to find differences in either direction |
| `UNION` | Only when you really need duplicate removal |
| Join hints | They fix the order *and* algorithm; they break when data changes |

### 9. Common mistakes
- Filtering the non-preserved side in `WHERE` → outer join becomes inner → move the condition into `ON`.
- `NOT IN` with NULLs → empty result → `NOT EXISTS`.
- Summing after a one-to-many join → inflated totals → aggregate before the join.
- Using `DISTINCT` to hide duplicates from a missing join condition → wrong and slow → fix the `ON` clause.
- `ORDER BY` inside each query of a `UNION` → error → one `ORDER BY` at the end.
- Mixing old-style comma joins with a forgotten `WHERE` → accidental cartesian product → use explicit `JOIN … ON`.

### 10. 🎯 Interview kit
**30-second answer:** A join matches rows from two tables on a predicate and combines their columns. Inner keeps matches; outer also keeps unmatched rows from the preserved side with NULLs. Set operators combine whole query results by column position: UNION ALL stacks, UNION also removes duplicates, INTERSECT keeps common rows, EXCEPT keeps rows only in the first. Set operators treat NULLs as equal; join predicates don't.

**Structure a longer answer:** logical steps (product → ON → outer rows) → ON vs. WHERE in outer joins → semi/anti joins and the NOT IN trap → physical algorithms (nested loops, merge, hash) → a real bug you fixed.

**Likely follow-ups:**
- *ON vs. WHERE in a LEFT JOIN?* → ON decides matching; WHERE filters afterwards and can remove outer rows.
- *Why does NOT IN return nothing?* → a NULL in the list makes every `<>` comparison UNKNOWN.
- *UNION vs. UNION ALL performance?* → UNION adds a distinct sort/hash.
- *When is a merge join chosen?* → both inputs sorted on the key, typically from indexes.

**Signals interviewers look for:** precise NULL reasoning; awareness of row multiplication; linking logical joins to physical operators and indexes.

### 11. Check yourself
- [ ] Write the three logical steps of an outer join and show where a WHERE filter cancels it.
- [ ] Demonstrate the NOT IN + NULL trap and fix it.
- [ ] Find rows that differ between two tables in both directions using EXCEPT.
- [ ] Explain when the optimizer chooses nested loops vs. hash match.
- [ ] Fix a report that double-counts after a join to order lines.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Set Operators: UNION, INTERSECT, EXCEPT
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › UNION Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › INTERSECT Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Example: Employee Union, Intersect, Except
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Conclusion
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions

