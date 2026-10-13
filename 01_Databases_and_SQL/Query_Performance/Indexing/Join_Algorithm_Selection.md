---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Join Algorithm Selection

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The optimizer joins tables with one of three physical algorithms — nested loops, hash, or merge — chosen from estimated row counts and available indexes, and the index on the join key is what makes nested loops (and merge) cheap.

### 1. The big picture
A logical `JOIN` says *what* rows pair up; the engine must decide *how*. Execution plans reveal the choice. Each algorithm has a sweet spot:

| Algorithm | Best when | Needs | Cost shape |
|---|---|---|---|
| **Nested loops** | One side small (or filtered to few rows) and the other side has an index on the join key | Index on inner side | rows(outer) × cost of one seek |
| **Hash join** | Large, unsorted inputs; equality join | Memory for a hash table (spills to disk otherwise) | rows(build) + rows(probe) |
| **Merge join** | Both inputs already sorted on the join key (indexes or earlier sort); equality or range | Sorted inputs | rows(left) + rows(right) |

Indexing the join key is therefore not one rule but three: it makes nested loops possible without scanning the inner table per row, it can provide sort order for a merge join, and it can let a hash join read a narrower structure. Most real design questions are about the **foreign key side** and **bridge tables**.

### 2. Core concepts and vocabulary
- **Outer / inner input** — in nested loops, the outer is read once; for each outer row the inner is probed.
- **Build / probe input** — in hash joins, the build side is hashed into memory; the probe side streams against it. The optimizer builds on the smaller input.
- **Join predicate / join key** — the column(s) in the `ON` clause.
- **Equi-join** — equality join; the only kind hash joins support.
- **Index nested loops** — nested loops where the inner side is an index seek.
- **Spill (to tempdb/disk)** — when a hash table or sort doesn't fit its memory grant.
- **Memory grant** — workspace memory the engine reserves for sorts/hashes; granted from estimates.
- **Broadcast hash join / shuffle hash / sort-merge (Spark)** — distributed analogues.
- **Bridge (junction) table** — resolves many-to-many (`user_roles(user_id, role_id)`).

### 3. How it works, step by step
**Nested loops (index):**
1. Read the outer input (e.g. 20 filtered customers).
2. For each outer row, seek into the inner index on the join key (e.g. `Orders(CustomerID)`).
3. Emit matches. Cost ≈ 20 seeks — excellent. With *no* usable index on the inner side each probe is a scan: 20 × table size, "terrible".

**Hash join:**
1. Build: read the smaller input, hash each row's join key into buckets in memory.
2. Probe: stream the larger input; hash its key, look into the bucket, compare, emit.
3. If memory is insufficient, partition both inputs to disk (Grace hash) and process partition pairs.

**Merge join:**
1. Both inputs sorted on the key (index order or explicit Sort).
2. Advance two cursors in lockstep, emitting matching groups. Single pass; low memory; many-to-many needs a worktable for duplicates.

**Optimizer decision** (simplified): estimate rows for each input after filters → cost the three algorithms with available indexes → pick the cheapest. Inaccurate estimates (stale stats, correlated predicates) lead to wrong choices: a nested loop planned for 10 rows that gets 10 million.

### 4. Worked example
```sql
-- 500M-row fact, 50-row dimension
SELECT d.region, SUM(f.amount)
FROM fact_sales f JOIN dim_store d ON d.store_id = f.store_id
WHERE f.sale_date >= '2025-01-01'
GROUP BY d.region;
-- Typical plan: Hash Join (build = dim_store: 50 rows, probe = fact scan). That's CORRECT:
-- probing the fact table's index 500M times would be far worse than one scan + a tiny hash.

-- Opposite case: small driving set, big indexed table
SELECT o.*
FROM customers c JOIN orders o ON o.customer_id = c.customer_id
WHERE c.email = 'a@x.com';
-- Needs: INDEX on orders(customer_id) -> Index Nested Loops: 1 customer seek + few order seeks.
-- Without the index: Hash Join over a full scan of orders, or nested loops scanning orders per customer.

CREATE INDEX ix_orders_customer ON orders (customer_id);
```
Read the plan: operator type, which input is on top (outer/build), actual vs estimated rows, `Memory Grant`/`Spill` warnings, and `Number of Executions` of the inner seek (a nested loop with a million executions is a red flag).

To *steer* when estimates are wrong, prefer fixing statistics and indexes; if you must: join hints (`OPTION (HASH JOIN)`, `LOOP JOIN`), PostgreSQL `SET LOCAL enable_hashjoin = off`, Spark `/*+ BROADCAST(dim) */` or `MERGE`/`SHUFFLE_HASH` hints.

### 5. Under the hood
- **Cost model.** Nested loops cost ∝ outer rows × (seek cost ≈ tree height + lookups); hash ∝ build + probe + possible spill; merge ∝ inputs + sort cost if not pre-sorted. The crossover is typically a few thousand outer rows with a good index; beyond it hash wins.
- **Why a hash join "materializes" the fact side.** If the planner thinks the filtered fact side is large (or misestimates), it hashes and scans rather than probing 500 M times. To prefer index nested loops with a small dimension driver, the *dimension* must be the outer input, the fact table must have an index on the FK, and estimated outer rows must be small — or reverse the roles (build the hash on the small dimension and stream facts, which is usually best for aggregates).
- **Foreign keys are not indexed automatically** (SQL Server, PostgreSQL; InnoDB does create one). Without a child-side index, `DELETE` on the parent must scan the child table to validate — the cited example went from 4,513 reads to 2 after indexing `SalesOrderHeader(CustomerID)` — and holds locks longer.
- **Bridge tables.** For `student_courses(student_id, course_id)`: primary key `(student_id, course_id)` serves "courses for a student"; add a second index `(course_id, student_id)` for "students in a course". One index only serves one direction because of the leftmost-prefix rule; `INCLUDE` or covering both directions avoids lookups.
- **Merge join prerequisites.** Both inputs sorted on the join key (plus equality or a monotonic range); great when two clustered indexes align on the key (e.g. `OrderHeader` and `OrderDetail` both clustered by `OrderID`) or when an `ORDER BY` on the join key is needed anyway.
- **Distributed engines.** Broadcast hash join: ship the small table to every executor (no shuffle). Shuffle hash / sort-merge: repartition both sides by key (expensive network). Spark picks broadcast under `spark.sql.autoBroadcastJoinThreshold` (default 10 MB); AQE can convert at runtime.
- **ORM effects.** An eager-loading `LEFT JOIN` for an optional relation can change cardinality estimates and plan; row multiplication also inflates results. Prefer separate keyed queries (`WHERE id IN (…)`) or ensure the join key is indexed and the select list narrow. N+1 (one query per parent) is worse; fix with a single join or batched `IN`.

### 6. Performance and design implications
- **Index the child-side FK** and any column used to look up dimension rows by business key during loads.
- **Small lookup/reference tables** (currency codes): always cheap, hash or loop either way; keep them cached, key narrow; index the join key anyway for planner stats and uniqueness.
- **Star schemas:** index fact FKs only if queries filter facts by dimension attributes with selective dimension filters (semi-join reduction via nested loop / bitmap); for full-fact aggregation, scans + hash joins dominate and columnstore/partitioning matter more.
- **Many-to-many:** two composite indexes (both directions) plus the PK.
- **Watch memory grants:** large hashes/sorts with underestimated rows spill; fix estimates or increase memory for the workload class.
- **Metrics:** actual rows, executions, spill events, logical reads per operator.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN (ANALYZE, BUFFERS)` / actual plan | Shows join type, loops, rows | Always |
| `OPTION (LOOP/HASH/MERGE JOIN)`, `INNER LOOP JOIN` | Force algorithm (SQL Server) | Last resort, testing |
| `SET enable_nestloop/hashjoin/mergejoin` (PG) | Disable an algorithm for session/transaction | Diagnosing plan choice (`SET LOCAL`) |
| `work_mem`, memory grant hints | Memory for hash/sort | Spills |
| `/*+ BROADCAST(t) */`, `MERGE`, `SHUFFLE_HASH` (Spark SQL) | Join strategy hints | Distributed joins |
| `CREATE INDEX … ON child (fk)` | FK support index | Always for child FKs |
| `UPDATE STATISTICS`, `ANALYZE` | Better row estimates | Plan instability |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Nested loops + index: few outer rows | Outer rows are large or underestimated |
| Hash: big unsorted equi-joins, analytics | Memory-constrained, skewed keys, non-equi join |
| Merge: pre-sorted inputs, range joins | Sorting both sides would be needed anyway |
| Hints to unblock production | Used instead of fixing statistics/indexes |

### 9. Common mistakes
- **Unindexed FK on the child table** → slow joins, slow parent deletes, lock waits → index it.
- **One index for a bridge table** → one direction fast, other scans → two composites.
- **Forcing a loop join** → catastrophic when data grows → fix estimates.
- **Join on mismatched types/collations** → implicit conversion, no seek → align types.
- **Joining on function results** → no index → persist/normalize the key.
- **Assuming hash join is "bad"** → it's the right plan for big joins; judge by estimate accuracy and spills.

### 10. 🎯 Interview kit
**30-second answer:** Nested loops probe the inner side once per outer row, so they win with few outer rows and an index on the join key. Hash joins build a hash table on the smaller input and probe with the larger, best for big unsorted equi-joins. Merge joins zip together two inputs already sorted on the key. Indexing join columns enables nested loops and ordered merges and avoids full scans.

**Structure a longer answer:** define join algorithms → cost shape → role of indexes → estimates and failure modes → how to read/steer plans → FK and bridge-table indexing.

**Likely follow-ups:**
- *Large fact joined to a tiny dimension?* → Hash with dimension as build is right; nested loop into fact would be 500 M probes.
- *When merge?* → Both sorted on key already, or range join, or ordered output needed.
- *ORM LEFT JOIN flipped the plan?* → Check estimates, index the join key, narrow the select, or split into keyed queries.
- *Why index foreign keys?* → Join performance and parent DELETE/UPDATE validation without scanning the child.
- *Bridge table?* → `(a,b)` PK and `(b,a)` secondary index.

**Signals interviewers look for:** you tie algorithm to cardinality and index availability; you mention estimates and spills; you propose measured, minimal steering.

### 11. Check yourself
- [ ] Describe the three join algorithms with cost formulas in words.
- [ ] Explain why a nested loop with no inner index is terrible.
- [ ] Design indexes for `user_roles` to serve both lookup directions.
- [ ] Identify in a plan a spill and a misestimated loop join.
- [ ] Explain how broadcast hash join differs from a shuffle join in Spark.
<!-- explainer:end -->

## Practice questions

- [Basic] [Your daily fact table load runs a lookup against a dimension table on business_key. Which column would you index and why?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20272_Your_daily_fact_table_load_runs_a_lookup_against_a_dimension.md>)
- [Intermediate] [In a star schema, why do you usually index the foreign keys in the fact table that point to each dimension?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20282_In_a_star_schema_why_do_you_usually_index_the_foreign_keys_i.md>)
- [Intermediate] [Explain how indexes help with JOIN operations.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19494_Explain_how_indexes_help_with_JOIN_operations.md>)
- [Intermediate] [You have a many-to-many join table student_courses(student_id, course_id). What indexes would you add and why?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20050_You_have_a_many-to-many_join_table_student_courses_student_i.md>)
- [Intermediate] [A query performs well until an ORM adds LEFT JOIN to eager-load an optional relation, flipping the plan. How do you keep eager-loading without defeating the primary index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19842_A_query_performs_well_until_an_ORM_adds_LEFT_JOIN_to_eager-l.md>)
- [Deep] [When is a merge join preferable, and what precondition on the inputs does it need?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19587_When_is_a_merge_join_preferable_and_what_precondition_on_the.md>)
- [Deep] [How would you index a query that needs the latest status per entity across millions of entities, comparing DISTINCT ON, a lateral join, and a current-state table?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20149_How_would_you_index_a_query_that_needs_the_latest_status_per.md>)
- [Deep] [A query does GROUP BY on an unindexed high-cardinality column and hashes in memory, spilling to disk. When does an index avoid the hash aggregate and when is the hash still better?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19836_A_query_does_GROUP_BY_on_an_unindexed_high-cardinality_colum.md>)

## Related concepts

- [EXISTS Semi-joins and Predicate Placement](<../../SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>) *(Joins and Subqueries)*
- [Query Rewrites and Plan Stability](<../Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Nested Loops vs. Hash vs. Merge Joins
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Nested Loops on Large Joins
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Wide Index Keys
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Hash Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Keys
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › Broadcast hash join (BHJ)

