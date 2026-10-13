---
id: "TOOLS/SQL_Server/04_Query_Processing/Execution_Plans/Join_Operators"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Join Operators

**L5 Expert** · [↑ Reading Execution Plans](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A logical join (INNER, LEFT, SEMI…) is carried out by one of three physical algorithms (Nested Loops, Merge Join or Hash Match), or by an Adaptive Join that picks between two of them at runtime. The optimizer chooses based on *estimated* row counts, sort order, available indexes and memory.

### 1. The big picture
When you write `JOIN`, you describe *what* rows should match (the **logical** join). You don't say *how* to find them. The optimizer picks a **physical join operator**. There are three classic algorithms, and each one is cheap in one situation and expensive in another:

| Algorithm | Best when | Cost grows with |
|---|---|---|
| **Nested Loops** | the outer (top) input is small and the inner (bottom) input has an index on the join key | outer rows × cost of one inner lookup |
| **Merge Join** | both inputs are large *and already sorted* on the join key | sum of both inputs (plus any Sort it needs) |
| **Hash Match** | both inputs are large, unsorted, with no useful index | sum of both inputs, plus memory for the hash table |

No algorithm is "the best." The optimizer costs each candidate using cardinality estimates (lesson *Statistics*) and keeps the cheapest. So a bad join choice is almost always a *symptom*. The real cause is usually a wrong estimate, a missing index, or a non-SARGable predicate. This lesson teaches you to read each operator, know when it is correct, and spot when it isn't.

### 2. Core concepts and vocabulary
- **Logical join**: the relational meaning: inner, left/right/full outer, cross, semi (`EXISTS`/`IN`), anti-semi (`NOT EXISTS`). It shows in the operator's *Logical Operation* property.
- **Physical join**: the algorithm: Nested Loops, Merge Join, Hash Match, Adaptive Join.
- **Outer input / inner input**: in a graphical plan, the **top** input of a join is the *outer* (Nested Loops) or *build* (Hash) input. The **bottom** input is the *inner* or *probe* input. These names have nothing to do with OUTER JOIN.
- **Equi-join**: a join on `=`. Hash and Merge need at least one equality predicate. Nested Loops can do any predicate (`<`, `BETWEEN`, `LIKE`, functions).
- **Build / probe**: the two phases of a hash join. Build a hash table from one input, then probe it with the other.
- **Residual predicate**: a join condition checked *after* the main match (e.g. a second non-equality condition). It shows as *Residual* in Hash/Merge properties.
- **Memory grant**: memory reserved before execution for hash tables and sorts (lesson *Memory Grants and Spills*).
- **Spill**: when a grant is too small, rows overflow to tempdb. The operator shows a warning.
- **Adaptive Join**: a batch-mode operator (2017+) that chooses Hash or Nested Loops after it has counted the rows from its first input.

### 3. How it works, step by step

**Nested Loops**
1. Read one row from the outer (top) input.
2. Run the inner (bottom) input *for that row*. Usually that means an Index Seek using the outer row's key value (the *Outer References* property names the column passed down).
3. Return every matching combination. For outer joins, return the outer row with NULLs if nothing matched.
4. Repeat for the next outer row.

Total work ≈ *outer rows × one inner execution*. With 10 outer rows and an index seek of 3 logical reads, that's about 30 reads, which is extremely cheap. With 2 million outer rows it's 6 million reads, which is a disaster. Check the inner operator's **Number of Executions** property: it equals the outer row count. Nested Loops needs no memory and no sorting, and it returns the first row immediately. That's why `TOP` and `FAST n` queries love it. Optimizations you'll see: **prefetching** (*WithUnorderedPrefetch / WithOrderedPrefetch*, asynchronous read-ahead of inner pages) and an **optimized** loop that sorts outer rows to make inner seeks more sequential.

**Merge Join**
1. Both inputs must arrive sorted on the join key, either from an index order or from an explicit **Sort** operator.
2. Read the current row from each side and compare the keys.
3. Equal → output the combination and advance. Top key smaller → advance top. Bottom key smaller → advance bottom.
4. Continue until either input ends. Each input is read **once**.

Merge is **one-to-many** when one side is known to be unique (e.g. a primary key). That's the common header→detail case. If neither side is unique it runs **many-to-many** and must save duplicate rows in a worktable in tempdb so it can "rewind." Check the *Many to Many* property: `True` means extra tempdb I/O. Merge needs no memory grant of its own, but any Sort feeding it does.

```
Top (sorted):    1  1  2  3            step: compare heads → equal → emit, advance the
Bottom (sorted): 1  2  2  3  4               side with the duplicate key; smaller → advance
```

**Hash Match**
1. **Build phase:** read the whole top (build) input. For each row, hash the join key and put the row in a bucket of an in-memory hash table. The optimizer puts the *smaller* estimated input on top.
2. **Probe phase:** read the bottom (probe) input row by row. Hash its key, go to the matching bucket, and compare the actual key values (two keys can share a hash value).
3. Output the matches. Outer and anti joins also track unmatched rows.
4. If the build input doesn't fit in the memory grant, partitions are written to tempdb and processed in later passes (*hash recursion*; in the worst case *hash bailout*). You'll see a **Hash Warning** / spill warning.

Hash is **blocking** on the build side: no row comes out until the whole build input has been read. It needs a memory grant sized from the *estimated* build rows × row size. It works on unsorted input and scales well, which makes it the workhorse of large reporting queries and of columnstore/batch mode.

**Adaptive Join** (batch mode; SQL Server 2017+, row-store tables from 2019 with compatibility level 150+)
1. The operator has three inputs: the build input, a Hash branch (scan) and a Nested Loops branch (seek).
2. It reads the build input and counts the rows.
3. If the count is ≥ the **Adaptive Threshold Rows** property, it runs the hash branch. Below the threshold, it uses the rows as the outer input of a nested loop that seeks into the inner index.
4. The actual plan shows *Estimated Join Type* and *Actual Join Type*, so you can tell which path ran.

In the example from the tuning book, the threshold was about 1,955 rows. 25,200 rows arrived, so the hash branch ran and the seek branch processed 0 rows. Adaptive joins protect against the "estimate said 10, reality was 100,000" problem in one specific, common place.

### 4. Worked example
```sql
-- AdventureWorks. Turn on actual plans (Ctrl+M) and I/O stats.
SET STATISTICS IO, TIME ON;

-- (A) Merge: header and detail both clustered on SalesOrderID → both sorted, large
SELECT soh.SalesOrderID, sod.OrderQty, sod.ProductID
FROM Sales.SalesOrderHeader AS soh
JOIN Sales.SalesOrderDetail AS sod ON soh.SalesOrderID = sod.SalesOrderID;

-- (B) Nested Loops: a selective filter makes the outer side tiny
SELECT soh.SalesOrderID, sod.OrderQty, sod.ProductID
FROM Sales.SalesOrderHeader AS soh
JOIN Sales.SalesOrderDetail AS sod ON soh.SalesOrderID = sod.SalesOrderID
WHERE soh.CustomerID = 11000;

-- (C) Hash: big inputs, join column (CustomerID) not the sort order of the
--     table that supplies TotalDue → no free ordering, no covering index
SELECT c.CustomerID, s.TotalDue
FROM Sales.Customer AS c
JOIN Sales.SalesOrderHeader AS s ON c.CustomerID = s.CustomerID;
```
How to read them:
- **(A)** Merge Join (Inner Join), two Clustered Index Scans, *Many to Many = False* (header is unique on SalesOrderID). No Sort operator, because the order came free from the clustered indexes.
- **(B)** An Index Seek on `IX_SalesOrderHeader_CustomerID` returns ~3 rows (top). For each one, a Clustered Index Seek on SalesOrderDetail runs (bottom: *Number of Executions = 3*). Reads drop from ~1,500 in (A) to a few dozen.
- **(C)** Hash Match: Customer (smaller) is the build input, SalesOrderHeader is the probe. Hover over the Hash Match to see *Memory Grant* in the root SELECT properties. If you add `TotalDue` to the CustomerID index (as an `INCLUDE`), the probe side becomes a narrow index scan already ordered by CustomerID, and the optimizer may switch to a Merge Join.

To test alternatives, force them **in a test only**:
```sql
... OPTION (LOOP JOIN);        -- all joins in the query become nested loops
... OPTION (HASH JOIN, MERGE JOIN); -- optimizer may use only these two
SELECT ... FROM a INNER LOOP JOIN b ON ...;  -- one join; also FIXES join order for the whole query!
```
The tuning book's demonstration: on a four-table query, forcing `OPTION (LOOP JOIN)` raised reads from 2,364 to 3,740 and made the query about 25% slower, because a small 17-row scan became a 701-row scan. You get exactly what you asked for, including the cost.

### 5. Under the hood
- **Costing.** For each join, the optimizer computes the outer cardinality, the inner cost per execution, any sort cost, and the hash memory/IO cost. The cheapest total wins. The **crossover point** where Nested Loops stops being cheaper than Hash is often only a few hundred to a few thousand outer rows. That's why a cardinality error by a factor of 100 flips the right choice.
- **Join order.** For *n* tables there are many possible orders and tree shapes. The optimizer explores a subset using transformation rules (commutativity, associativity), within its time budget. A join hint written inline (`INNER HASH JOIN`) also *forces the written order* (an implicit `FORCE ORDER`). That's a big side effect people forget.
- **Hash internals.** The build side should be the smaller input. If estimates are wrong and the "small" side is actually huge, the table won't fit: *role reversal* and recursive partitioning to tempdb follow. *Bitmap* operators (in parallel and batch plans) build a filter from the build keys and push it down to the probe-side scan to discard non-matching rows early.
- **Merge internals.** Sorted input can come from an index, from a previous Merge/Stream Aggregate, or from a Sort. When the optimizer adds a Sort only to enable a Merge, compare the Sort cost against a Hash alternative. Many-to-many merges use a tempdb worktable (`Worktable` appears in `STATISTICS IO` output).
- **Nested Loops variants.** *Apply* (correlated) nested loops pass outer values into the inner side via **Outer References**. That's how `CROSS APPLY`, lookups (Key Lookup is itself a nested-loop join to the clustered index) and correlated subqueries run. A **Lazy Spool** on the inner side caches results for repeated outer values.
- **Batch mode** (columnstore, and rowstore in 2019+): hash joins process ~900-row batches with SIMD-friendly code, and they enable adaptive joins and memory grant feedback.
- **Parallelism.** Parallel hash joins repartition both inputs on the join key (*Repartition Streams*) or broadcast a small build input to all threads. Skewed keys give one thread most of the work.

### 6. Performance and design implications
- **Nested Loops with a huge outer input** (millions of executions on the inner seek) is a classic "estimate was too low" pattern. Compare *Estimated Number of Rows* vs *Actual* on the outer side, then fix statistics, parameter sniffing, or the predicate.
- **Hash Match with a tiny outer input** wastes a memory grant and blocks streaming. That usually means the estimate was too high, or an index on the join key is missing.
- **Index foreign keys.** A nonclustered index on the child table's FK column makes Nested Loops seeks possible, and gives ordered input for Merge.
- **Cover the query.** In example (C), a covering index changed which join types were possible.
- **Watch for spills** (yellow warning on Hash Match / Sort) and *Many to Many = True* merges. Both cost tempdb I/O.
- **Measure with:** actual execution plans, `SET STATISTICS IO` (reads per table, worktables), `sys.dm_exec_query_stats` (`total_spills`, grant columns), Query Store, and Extended Events `hash_warning` / `sort_warning`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTION (LOOP JOIN / MERGE JOIN / HASH JOIN)` | restricts the allowed physical joins for all joins in the query | testing alternatives; rare last resort |
| `INNER LOOP JOIN` (inline hint) | forces one join's algorithm **and** the whole join order | almost never in production |
| `OPTION (FORCE ORDER)` | keeps the written join order | diagnosing join-order problems |
| `OPTION (USE HINT('DISABLE_BATCH_MODE_ADAPTIVE_JOINS'))` | turns off adaptive joins for the query | an adaptive join misbehaves |
| `ALTER DATABASE SCOPED CONFIGURATION SET BATCH_MODE_ADAPTIVE_JOINS = OFF` | the same, database-wide | regression testing |
| `SET STATISTICS IO ON` | logical reads per table, plus `Worktable` | comparing join choices |
| Plan properties: *Number of Executions*, *Outer References*, *Many to Many*, *Adaptive Threshold Rows*, *Actual Join Type* | show how the join really ran | every plan review |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Nested Loops**: small outer input, indexed inner, OLTP, `TOP`/`EXISTS`, non-equality joins | outer input is large or underestimated; no index on the inner side |
| **Merge**: both inputs large and already ordered (clustered PK/FK), result needed in key order | an explicit Sort must be added for big inputs; many-to-many duplicates |
| **Hash**: large unsorted inputs, data warehouse queries, batch mode | memory pressure, underestimated build side (spills), need for first rows fast |
| **Adaptive**: row counts vary a lot between executions (parameter-sensitive) | not batch mode; low compat level; very unstable thresholds |
| **Hints**: proving a theory in test | as a permanent fix. They freeze one choice while the data keeps changing |

### 9. Common mistakes
- Treating "Hash Match = bad, Nested Loops = good" as a rule → you "fix" a correct plan → you get worse performance. Always compare estimates with actuals first.
- Adding `OPTION (LOOP JOIN)` to fix one join → every join in the query is forced → reads go up (the 2,364 → 3,740 example). Fix the estimate or the index instead.
- Using an inline join hint without realizing it fixes join order → a fragile plan that breaks as data grows → use query-level hints or Query Store hints, and only after testing.
- Missing index on a foreign key column → hash joins and scans on every parent→child query → index the FK.
- Reading the *top/bottom* inputs the wrong way round → misdiagnosis → top = outer/build, bottom = inner/probe.
- Ignoring spill warnings on Hash Match → tempdb I/O and slow, unpredictable runtimes → update statistics, fix estimates, rely on memory grant feedback.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server has three physical join algorithms. Nested Loops runs the inner input once per outer row. It's great for small outer sets with an index seek on the inner side. Merge Join walks two inputs sorted on the join key in one pass. It's great when both are big and already ordered by an index. Hash Match builds an in-memory hash table from the smaller input and probes it with the larger one. It's great for big unsorted sets but needs a memory grant and can spill. In batch mode, an Adaptive Join decides between hash and loops at runtime from the actual row count. The optimizer chooses based on estimates, so a bad join choice usually points to a bad estimate or a missing index.
**Structure a longer answer:** clarify (OLTP vs reporting, sizes) → explain each algorithm's mechanics and cost formula → how to read them in a plan (executions, many-to-many, spills, threshold) → why the choice goes wrong (cardinality) → fix the cause, not with hints → close with a scenario (e.g. a nested loop running 2M seeks because a parameter was sniffed for a rare value).
**Likely follow-ups:**
- *Which input is the build side?* → the top one; the optimizer tries to make it the smaller one.
- *Can a hash join do `a.x < b.y`?* → No. It needs an equality; nested loops can handle any predicate.
- *Why does a Merge Join sometimes have a Sort before it?* → the input wasn't already ordered; check whether the Sort costs more than a hash would.
- *What is a Key Lookup, as a join?* → a nested-loop join from a nonclustered index to the clustered index.
- *What does an Adaptive Join need?* → batch mode (columnstore, or batch mode on rowstore with compatibility level 150+), equi-join, and an index seek alternative.
**Signals interviewers look for:** you link join choice to cardinality estimates, you know the memory/tempdb implications, you read *Number of Executions* and *Actual vs Estimated* rows, and you're wary of hints.

### 11. Check yourself
- [ ] Draw the three algorithms for a 5-row and a 5-million-row outer input and estimate the work of each.
- [ ] Run queries (A)–(C) and identify the outer/inner (build/probe) inputs in each plan.
- [ ] Find the *Number of Executions* on the inner side of a Nested Loops and explain the value.
- [ ] Force `OPTION (LOOP JOIN)` on a multi-join query and compare `STATISTICS IO` before and after.
- [ ] Explain when a Merge Join runs many-to-many and what that costs.
- [ ] Describe how an Adaptive Join decides its branch and where to see which one ran.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Nested Loops vs. Hash vs. Merge Joins
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Join Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Nested Loops on Large Joins
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Nested Loop
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Detecting Inefficient Joins
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Hash Match
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Thinking About Performance
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Joins

