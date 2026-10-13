---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL"]
---

# Views and Materialized Views

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A view is a saved query that is re-run every time you read it; a materialized view stores the query's result so reads are fast, at the cost of staleness or maintenance work.

### 1. The big picture
Views and materialized views both give a name to a query. They differ in *when the work is done*. A plain view does the work at **read time**: it is just stored text that the optimiser expands into the calling query. A materialized view (indexed view in SQL Server) does the work at **write or refresh time** and keeps the result physically, trading storage and maintenance for read speed.

Views serve **abstraction and security** (hide joins, expose only some columns or rows, give a stable interface to a changing schema: the "external" level in the three-schema architecture). Materialized views serve **performance** (pre-computed aggregates and joins for reporting). Choosing between them is a question of how fresh the data must be and how read-heavy the workload is.

### 2. Core concepts and vocabulary
- **View** — a named, stored `SELECT`; has no data of its own.
- **Materialized view** — a stored result set derived from a query, with a refresh policy.
- **Indexed view (SQL Server)** — a view with a unique clustered index; the engine maintains it synchronously on every base-table change.
- **Refresh** — recomputing a materialized view: **complete** (rebuild), **incremental / fast** (apply changes), **on commit**, **on demand**, **scheduled**.
- **Staleness** — how far the stored result lags behind base tables.
- **Updatable view** — a view through which `INSERT/UPDATE/DELETE` reach a base table (simple single-table views; others need `INSTEAD OF` triggers).
- **View expansion (inlining)** — the optimiser substitutes the view definition into the outer query and optimises it as a whole.
- **`WITH CHECK OPTION`** — rejects modifications through the view that would make the row disappear from the view.
- **Query rewrite** — the optimiser transparently answers a query from a matching materialized view (Oracle, Enterprise-edition SQL Server).

### 3. How it works, step by step
**Plain view:**
1. `CREATE VIEW v AS SELECT …` stores the definition in the catalog.
2. A query `SELECT … FROM v WHERE …` is parsed; the view is replaced by its definition.
3. The optimiser plans the combined query (predicate pushdown, join elimination, etc.). There is no separate "view execution".

**Materialized view:**
1. `CREATE MATERIALIZED VIEW` runs the query once and stores the rows (often with indexes).
2. Reads hit the stored rows directly — cost independent of base-table size.
3. A **refresh** brings it up to date:
   - *Complete:* re-run the query, replace contents (Postgres `REFRESH MATERIALIZED VIEW`; `CONCURRENTLY` keeps it readable but requires a unique index).
   - *Incremental:* apply deltas from change logs (Oracle fast refresh with materialized view logs) or maintained synchronously (SQL Server indexed views).
4. Readers get a result as of the last refresh.

Comparison:

| | View | Materialized view |
|---|---|---|
| Stores data | No | Yes |
| Read cost | Full query cost | Cheap |
| Write cost | None | Maintenance (sync) or refresh job |
| Freshness | Always current | As of last refresh (unless sync) |
| Storage | None | Result size + indexes |
| Typical use | Abstraction, security | Aggregates, expensive joins |

### 4. Worked example
```sql
-- plain view: abstraction + column/row security
CREATE VIEW dbo.v_active_customers
AS
SELECT customer_id, name, city
FROM   dbo.Customers
WHERE  is_active = 1
WITH CHECK OPTION;       -- when used in a updatable path

-- SQL Server indexed view: synchronously maintained aggregate
CREATE VIEW dbo.v_daily_sales
WITH SCHEMABINDING
AS
SELECT order_date,
       COUNT_BIG(*)            AS order_cnt,
       SUM(amount)             AS total_amount   -- amount must be NOT NULL
FROM   dbo.Orders
GROUP BY order_date;
GO
CREATE UNIQUE CLUSTERED INDEX ix_v_daily_sales ON dbo.v_daily_sales(order_date);

-- PostgreSQL materialized view with scheduled refresh
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT order_date, COUNT(*) AS order_cnt, SUM(amount) AS total_amount
FROM   orders GROUP BY order_date;
CREATE UNIQUE INDEX ON mv_daily_sales(order_date);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```
How to read it: `SCHEMABINDING` prevents base-table changes that would break the view; `COUNT_BIG(*)` is mandatory in an aggregated indexed view so the engine can maintain groups on delete. Check the plan of a query against `v_daily_sales`: a seek on the view's clustered index means you are reading stored rows; on Standard edition you may need `WITH (NOEXPAND)` to force it. In Postgres, compare `EXPLAIN ANALYZE` of the aggregate against the base table versus `SELECT * FROM mv_daily_sales`.

### 5. Under the hood
- **Plain views are inlined.** Nested views multiply: a view over a view over a view can produce a plan with many redundant joins that the optimiser may not fully simplify; compile time and plan quality suffer.
- **Indexed views in SQL Server** are physically a clustered B-tree. Every `INSERT/UPDATE/DELETE` on base tables also updates the view inside the same transaction, which adds write cost and lock contention (aggregate groups become hot rows). Many restrictions apply: deterministic expressions, no outer joins, no `MIN/MAX` in aggregates (older versions), `SCHEMABINDING`.
- **Postgres materialized views** are not automatically maintained; the refresh is a manual or scheduled command. `CONCURRENTLY` builds a new copy and diffs it, allowing readers during refresh.
- **Oracle** uses materialized view logs to capture row changes for fast refresh, and the optimiser performs *query rewrite* when the stored result can answer a query.
- **Warehouses/lakehouses** (Snowflake, BigQuery, Databricks) offer managed materialized views or dynamic tables that refresh incrementally in the background, billed as compute.
- **Security.** A view can restrict columns/rows and be granted without granting the base table (ownership chaining in SQL Server). It is not a security boundary against side-channels (error messages, function leakage) unless defined as security-barrier/`SECURITY_BARRIER` where available.

### 6. Performance and design implications
- A view is not a performance feature; it can even hide cost, since consumers don't see the joins behind it.
- Materialized views shift cost from many reads to one refresh: worthwhile when reads ≫ writes and the aggregation is expensive.
- Keep refresh windows aligned with the freshness the business tolerates; a nightly refresh for a dashboard viewed hourly is fine.
- Index the materialized view like a table.
- Beware correctness: users may assume a materialized view is current; surface the refresh time.
- Measure: plan cost with and without, refresh duration, write overhead on base tables (for synchronous maintenance), storage size.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE VIEW / ALTER VIEW` | Define a view | Abstraction/security |
| `WITH SCHEMABINDING` | Locks view to base schema | Required for indexed views |
| `CREATE UNIQUE CLUSTERED INDEX` on a view | Materializes it (SQL Server) | Aggregation acceleration |
| `WITH (NOEXPAND)` | Forces use of indexed view | Non-Enterprise editions |
| `CREATE MATERIALIZED VIEW` / `REFRESH [CONCURRENTLY]` | Postgres materialization | Reporting snapshots |
| `WITH CHECK OPTION` | Prevent invisible modifications | Updatable filtered views |
| `INSTEAD OF` trigger | Makes complex views updatable | Legacy interface shims |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plain view for reuse, naming and access control | You expect it to be fast by itself |
| Materialized view for heavy aggregates read often | Data must be real-time and writes are heavy |
| Indexed view when you need always-current aggregate | Base tables have high write contention |
| Scheduled refresh with concurrently option | Refresh time exceeds the refresh interval |
| Summary table maintained by pipeline | Need engine-level query rewrite |

### 9. Common mistakes
- Nesting many views → huge plans and slow compiles → flatten or use inline table-valued functions.
- Using `SELECT *` in a view → columns change meaning or break after schema change → list columns; use `SCHEMABINDING`.
- Putting `ORDER BY` in a view → not guaranteed to order results → order in the outer query.
- Forgetting to refresh → stale dashboards → monitor refresh timestamps.
- Indexed view on a hot OLTP table → write slowdowns and deadlocks → move to a replica/pipeline.
- Assuming modifications through any view work → errors → know the updatability rules.

### 10. 🎯 Interview kit
**30-second answer:** A view is a stored query expanded at read time: no data stored, always current, good for abstraction and security. A materialized view stores the result, so reads are fast but it's stale until refreshed or costs write overhead if maintained synchronously. Use views for simplicity and security, materialized views for expensive, frequently read aggregates.

**Structure a longer answer:** definition → execution model (inlined vs stored) → refresh strategies → trade-offs (freshness, storage, write cost) → scenario (daily sales dashboard).

**Likely follow-ups:**
- *Can you update through a view?* → If it maps simply to one base table; otherwise `INSTEAD OF` trigger.
- *Do views improve performance?* → No; the query still runs unless materialized.
- *How do you keep a materialized view fresh?* → Scheduled/on-commit/incremental refresh or synchronous maintenance.
- *What is an indexed view?* → SQL Server's materialized view maintained synchronously.

**Signals interviewers look for:** you explain execution (inlining) rather than just definitions, and weigh freshness against cost.

### 11. Check yourself
- [ ] Explain what the optimiser does with a plain view.
- [ ] List three refresh strategies for a materialized view and their staleness.
- [ ] Create an indexed view and state its restrictions.
- [ ] Decide between a view, materialized view and summary table for a dashboard.
- [ ] Explain why `ORDER BY` in a view is not a guarantee.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is a view in SQL and what are its advantages?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22179_What_is_a_view_in_SQL_and_what_are_its_advantages.md>)
- [Basic] [What is a Data Definition Language and give three examples of DDL statements?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22357_What_is_a_Data_Definition_Language_and_give_three_examples_o.md>)
- [Basic] [What is the role of a Database Administrator and how is it different from a database developer?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22442_What_is_the_role_of_a_Database_Administrator_and_how_is_it_d.md>)
- [Intermediate] [What is a self join and when would you use one?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22203_What_is_a_self_join_and_when_would_you_use_one.md>)
- [Intermediate] [Why did you choose a relational database over a NoSQL store for your project?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22243_Why_did_you_choose_a_relational_database_over_a_NoSQL_store_.md>)
- [Intermediate] [What is a database role and how does it simplify managing permissions for many users?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22427_What_is_a_database_role_and_how_does_it_simplify_managing_pe.md>)
- [Deep] [What does EXPLAIN / EXPLAIN ANALYZE tell you about a query?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22208_What_does_EXPLAIN_EXPLAIN_ANALYZE_tell_you_about_a_query.md>)
- [Deep] [What is a wide-column store like Cassandra good at, and why is its data model query-first?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22328_What_is_a_wide-column_store_like_Cassandra_good_at_and_why_i.md>)

## Related concepts

- [SQL vs NoSQL](<../../SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>) *(SQL Fundamentals)*
- [Modeling Relationships and Schemas](<../../Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Using Views Effectively](<../../Database_Objects/Views/Using_Views_Effectively.md>) *(Views)*

## References

- *The Enterprise Big Data Lake* — Alex Gorelik — Historical Perspective › Storing and Querying the Data
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Data Management › SQL Analytics
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Unit testing interview questions › Database Fundamentals
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › Essential SQL for Data Engineers
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Summary
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Powerful query capabilities
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Database as Container
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Derived Tables
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › External Tables

