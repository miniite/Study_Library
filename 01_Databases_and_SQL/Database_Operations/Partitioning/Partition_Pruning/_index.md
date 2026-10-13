---
id: "01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Partitioning"
tools: []
---

# Partition Pruning

**L4 Advanced** · [↑ Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partition pruning is the optimizer skipping every partition whose bounds cannot satisfy the query's predicate, and it only works when the predicate filters the partition key in a form the optimizer can compare against those bounds.

### 1. The big picture
Partitioning pays off mostly through pruning (also called partition elimination). If a table has 60 monthly partitions and a query asks for one month, a pruned plan opens one partition; an unpruned plan opens 60 and is slower than if the table had never been partitioned, because it also pays per-partition overhead.

Pruning is a *planner* feature, so it depends on what the query text looks like, how parameters are supplied, and what the optimizer can prove at plan time or at run time. Most "partitioning didn't help" stories are pruning failures: the key is wrapped in a function, implicitly converted, or missing from the predicate. This lesson covers how pruning works, how to verify it, and how to debug it. It builds on the Partitioning Strategy lesson.

### 2. Core concepts and vocabulary
- **Static (plan-time) pruning** — partitions are eliminated while the plan is built, because the predicate uses constants the planner can see.
- **Dynamic (run-time) pruning** — the plan keeps all partitions as candidates and skips some at execution, once parameter values or join results are known.
- **Sargable predicate** — a condition of the form `key <op> value` where the key column is bare, so it can be compared with partition bounds (or used for an index seek).
- **Partition key** — the column(s) used to define the bounds.
- **Implicit conversion** — the engine casting the column (or the value) to another type to compare them; casting the column kills pruning.
- **Parameter sniffing** — using the parameter values seen on first execution to build a cached plan.

### 3. How it works, step by step
1. **Parse** the WHERE clause into conjuncts (predicates joined by AND).
2. **Match** each conjunct against the partition key. Only predicates that compare the bare key to a constant, parameter, or an expression evaluable without the table are usable: `=`, `<`, `<=`, `>`, `>=`, `BETWEEN`, `IN (…)`.
3. **Intersect** the implied value interval with the partition bounds. For range partitioning this is an interval overlap test; for list partitioning a membership test; for hash partitioning only equality (or an IN list) can be hashed, so ranges select everything.
4. **Emit the plan** containing only surviving partitions (static), or containing a "pruning step" that is resolved at execute time (dynamic) for parameters, prepared statements, nested loop joins, and subquery results.
5. **Execute**, scanning only surviving partitions and their local indexes.

Predicates joined by OR across different columns, or with the key hidden inside an expression, usually fall back to "all partitions".

### 4. Worked example
```sql
-- orders partitioned monthly by created_at (timestamp)
-- Good: bare key, constants → static pruning, 1 partition
EXPLAIN SELECT count(*) FROM orders
WHERE created_at >= '2026-09-01' AND created_at < '2026-10-01';

-- Bad: function on the key → no pruning, all partitions
EXPLAIN SELECT count(*) FROM orders
WHERE DATE(created_at) = '2026-09-15';

-- Rewrite: same meaning, sargable
EXPLAIN SELECT count(*) FROM orders
WHERE created_at >= '2026-09-15' AND created_at < '2026-09-16';
```
How to read the result: in PostgreSQL `EXPLAIN` lists only the partitions it will scan (e.g. one `Seq Scan on orders_2026_09`); with parameters, `EXPLAIN ANALYZE` shows `Subplans Removed: 11` and nodes marked `(never executed)`. In MySQL, `EXPLAIN` has a `partitions` column naming the partitions touched. In Oracle, the plan columns `Pstart`/`Pstop` show `KEY` for dynamic pruning, numbers for static, and `1`–`N` for all partitions. In SQL Server, the actual plan's scan operator shows "Actual Partition Count" and "Partitions Accessed"; a `Partition ID` seek predicate confirms elimination.

Why the function breaks it: the partition bounds describe values of `created_at`. `DATE(created_at)` is a different value space, and the planner would have to apply the function to every partition's boundaries (and prove monotonicity) to use it. Most engines don't, so they conservatively scan everything.

### 5. Under the hood
- **Bound comparison happens on the stored key type.** If the column is `timestamp` and the literal is typed `timestamptz`, or the column is `varchar` and the parameter is `nvarchar`, the conversion can land on the column side (in SQL Server, data-type precedence converts the lower-precedence side, often the column), which disables elimination and index seeks. Always match types precisely.
- **Simple parameterization.** SQL Server can auto-parameterize literals into `@1`; the plan is then built without knowing the value, which can limit elimination to run time or (for some forms) prevent it. Fixes: write queries so the key filter is compatible with partition elimination, avoid forcing conversions, and in some cases add `OPTION (RECOMPILE)` so the optimizer sees the literal.
- **Bind variables vs. literals.** Literals allow static pruning and the best estimates but create many plans. Bind variables allow reuse and still prune at run time in modern engines (PostgreSQL's generic plans use run-time pruning; Oracle uses `KEY` pruning). The catch is parameter sniffing: a cached plan built for a single-partition value may be wrong for a wide range, so test with both narrow and wide parameters.
- **Joins.** If the partition key comes from the *other* table rather than a filter, only dynamic pruning can help: the optimizer pushes values from the driving side into the partitioned side (nested loop probing, or a runtime filter / "dynamic partition pruning" in Spark and warehouse engines). Without it the partitioned table is scanned in full.
- **Partition-wise combos.** After pruning, remaining partitions can be scanned in parallel and aggregated independently (partition-wise aggregation), then the partial results merged.
- **Stale or missing statistics** don't change which partitions are eliminated (that is bound-based), but they change cost estimates and thus join and index choices on the surviving partitions.

### 6. Performance and design implications
- **Measure pruning, not just time.** Metrics that tell the real story: partitions scanned vs. total, logical reads / buffers hit, rows read vs. rows returned, and planning time. A "scan reduction ratio" = 1 − scanned/total is easy to track.
- **Pair pruning with an index inside the partition.** Pruning narrows to a partition; a local index (ideally covering) narrows within it. A hot dashboard query over "last 24 hours for tenant X" can become a single-partition, index-only seek.
- **ORMs and generated SQL.** ORMs often emit `CAST(created_at AS date)`, `LOWER(col)`, or bind a string where the column is a date. You may not control thousands of such queries. Mitigations: add a view or computed key that matches the generated expression, fix type mappings in the ORM configuration centrally, or partition on the expression the ORM really uses (generated column/computed column), accepting that this is a design trade-off.
- **Small tables show nothing.** Pruning benefits are only visible when the table is large enough for I/O to dominate; benchmark with realistic volume.
- **Planning cost.** With thousands of partitions, even a perfectly pruned query spends time in planning; check planning time separately from execution time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN` / `EXPLAIN ANALYZE` (PG) | shows partitions kept and `never executed` nodes | confirm pruning |
| `EXPLAIN PARTITIONS` / `partitions` column (MySQL) | lists partitions touched | confirm pruning |
| Plan columns `Pstart`/`Pstop` (Oracle) | static vs. `KEY` vs. all | read pruning in plans |
| Actual plan, Partition Count (SQL Server) | partitions accessed | verify elimination |
| `enable_partition_pruning` (PG) | master switch | diagnosing, never leave off |
| `plan_cache_mode`, `OPTION (RECOMPILE)` | control generic vs. custom plans | parameter-sensitive pruning |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rewriting predicates to be sargable (`>= x AND < y`) | Rewrite changes semantics at boundary (inclusive/exclusive end) |
| Using literals/RECOMPILE for critical queries | Query is executed thousands of times per second (compile cost) |
| Using bind variables with run-time pruning | Data skew makes one cached plan wrong for other parameter values |
| Computed/generated partition key for ORM predicates | Extra column and write overhead you can avoid by fixing the ORM |

### 9. Common mistakes
- Wrapping the key in a function (`DATE(ts)`, `YEAR(d)`, `CAST`) → all partitions scanned → compare the bare column with a range.
- Type mismatch between parameter and column → implicit conversion on the column → fix data types in the client and ORM mapping.
- Filtering on another column correlated with the key (e.g. `order_id`) → optimizer can't know the relationship → include the key predicate too.
- Using hash partitioning and then issuing range queries → no pruning → use range partitioning, or only equality lookups.
- Judging by elapsed time only → a warm cache hides a full-partition scan → look at partitions scanned and reads.
- Testing on tiny data or one parameter value → plan looks fine in test, differs in production (parameter sniffing) → test narrow and wide parameters on production-sized data.

### 10. 🎯 Interview kit
**30-second answer:** Pruning is the planner eliminating partitions whose bounds can't match the filter on the partition key. It works with bare-key sargable predicates, statically for constants and dynamically for parameters and joins. You verify it in the plan by looking at which partitions are scanned.

**Structure a longer answer:** clarify engine and partition method → explain bound comparison → name what breaks it (functions, conversions, missing key predicate, hash with ranges) → how to verify in plans → how to fix, including ORM and parameter cases → trade-offs.

**Likely follow-ups:**
- *Why does `DATE(created_at)` break pruning?* → the predicate is on a derived value; the planner can't map it to the stored bounds; rewrite as a range.
- *Filter on the key but all partitions are scanned — debug?* → check the exact predicate text, data types and implicit conversions, parameterization, whether pruning is enabled, and whether the filter is inside an OR or a subquery.
- *Bind variables vs. literals?* → run-time pruning works for binds, but sniffing can pick a plan fitted to one value; literals give static pruning and more plans.
- *Static vs. dynamic pruning?* → plan time vs. execute time; dynamic matters for parameters, prepared statements, and join-driven keys.
- *Join where the key comes from the other table?* → needs dynamic pruning or an explicit filter.

**Signals interviewers look for:** you verify with plans rather than assume; you know the sargability rule; you can reason about types, parameters and ORMs; you combine pruning with local indexes.

### 11. Check yourself
- [ ] Explain static vs. dynamic pruning with one example of each.
- [ ] Rewrite `WHERE YEAR(order_date) = 2026` so it prunes.
- [ ] Show in a plan of your engine of choice how many partitions were touched.
- [ ] Explain why a hash-partitioned table can't prune `BETWEEN` predicates.
- [ ] Describe how a parameter type mismatch can silently disable pruning.
- [ ] Propose a fix for an ORM that always wraps the key in `CAST`.
<!-- explainer:end -->

## Go deeper

- [Partitioning Strategy](<Partitioning_Strategy.md>)

## Practice questions

- [Intermediate] [How does partition pruning work, and how would you confirm from an execution plan that only the intended partitions were scanned?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-20972_How_does_partition_pruning_work_and_how_would_you_confirm_fr.md>)
- [Intermediate] [Why can partition pruning fail when you wrap the partition key in a function like DATE(created_at) in the WHERE clause?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-20989_Why_can_partition_pruning_fail_when_you_wrap_the_partition_k.md>)
- [Deep] [A query that filters on the partition key still scans every partition. Walk me through how you would debug why pruning didn't kick in.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-20973_A_query_that_filters_on_the_partition_key_still_scans_every_.md>)
- [Deep] [Explain how partition pruning behaves with bind variables / prepared statements versus literal values.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21023_Explain_how_partition_pruning_behaves_with_bind_variables_pr.md>)
- [Deep] [Explain how you'd design partitioning + a materialized view together so aggregate queries stay fast as data grows.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21043_Explain_how_you_d_design_partitioning_a_materialized_view_to.md>)
- [Deep] [How do you benchmark partition pruning effectiveness — what metrics or plan attributes tell the real story?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21047_How_do_you_benchmark_partition_pruning_effectiveness_what_me.md>)
- [Deep] [How would you use partition pruning together with a covering index to make a hot dashboard query sub-10ms?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21054_How_would_you_use_partition_pruning_together_with_a_covering.md>)
- [Deep] [How do you architect partition pruning to survive an ORM that generates function-wrapped or implicitly-cast predicates across thousands of queries you don't control?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21097_How_do_you_architect_partition_pruning_to_survive_an_ORM_tha.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Partitioning and Data Skipping in Lakes](<../../../Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*

