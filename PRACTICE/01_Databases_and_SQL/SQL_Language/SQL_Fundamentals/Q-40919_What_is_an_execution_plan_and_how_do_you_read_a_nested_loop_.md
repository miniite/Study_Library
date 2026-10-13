---
type: "practice"
difficulty: "Expert"
---

# What is an execution plan, and how do you read a nested loop vs hash join vs merge join?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Two things. (1) What is an execution plan and how do I read one? (2) The three physical join operators — nested loop, hash join, merge join — how each works, how to recognize it in a plan, and when the optimizer picks each.

What the interviewer (expertise level) is probing: Can I go beyond "run EXPLAIN"? They want the mental model of a plan as a tree of physical operators with estimated vs actual rows and costs, the ability to spot the usual pathologies (bad cardinality estimates, spills, the wrong join for the data size), and the judgment to know why the optimizer chose an operator and how to nudge it. This is a diagnosis-under-pressure question.

#### Assumptions
/ clarifying questions: Engine-agnostic core (Postgres/SQL Server/Oracle/MySQL all expose these), but I'll use Postgres EXPLAIN (ANALYZE, BUFFERS) and SQL Server terminology since they're the most explicit. I'll assume the goal is diagnosing a slow join query.

Plan: (1) define an execution plan + how to read the tree, (2) the three joins mechanically with cost, memory, and ordering requirements, (3) how each appears in a plan and what triggers it, (4) the diagnostic playbook and common traps, (5) summary.

1. What an execution plan is and how to read it

A SQL query is declarative — you say what you want, not how. The query optimizer translates it into a physical execution plan: a tree of operators (scans, joins, sorts, aggregates) that the engine will actually run. Each operator consumes rows from its children and emits rows to its parent. Execution flows leaves → root (children produce, parents consume), even though we read the printout top-down.

How to actually read it:

EXPLAIN shows the estimated plan (costs and row counts the optimizer guessed). EXPLAIN ANALYZE (Postgres) / Actual Execution Plan (SQL Server) runs the query and adds actual rows, loops, and timing.
For every operator note four things: estimated rows vs actual rows, cost/time, number of executions (loops), and memory/spills.
The single most important diagnostic: compare estimated rows to actual rows. A big divergence (e.g., estimate 10, actual 2,000,000) means the optimizer had bad statistics and almost certainly picked the wrong join/access method downstream. Fix the estimate (update stats, better predicates, remove functions-on-columns) before blaming the operator.
Read from the most-indented / rightmost operator inward — those are the leaves that run first. The Rows and Buffers/logical reads tell you where the I/O and time concentrate.
Watch for red flags: Seq Scan / Table Scan on a large table where an index was expected; Sort or Hash spilling to disk (external merge Disk in Postgres, Sort Warnings/tempdb spill in SQL Server); an operator with loops=N where N is huge (a nested loop over too many outer rows); and any Filter discarding most rows after a scan (predicate not pushed to an index).
2. The three physical joins
Nested Loop Join

Mechanic: for each row of the outer input, probe the inner input for matches — typically via an index seek. Cost ≈ outer_rows × cost_of_inner_probe.

Best when: the outer input is small and the inner side has a selective index on the join key. Then each probe is an O(log n) index seek and the total is cheap.
Terrible when: the outer is large and the inner has no usable index → it degenerates to an O(m×n) scan-per-row.
Memory: minimal — no build phase, streams results, low startup cost (good for LIMIT/TOP and OLTP point lookups).
In the plan: Nested Loop (Postgres/SQL Server). Look at the inner operator's loops count — that's how many times it ran.
Hash Join

Mechanic: build phase — read the smaller ("build") input fully and construct an in-memory hash table keyed on the join column. Probe phase — stream the larger input, hashing each row's key to look up matches. Cost ≈ build_rows + probe_rows (roughly linear).

Best when: joining two large, unsorted inputs on an equality predicate, with no useful index on the join keys. This is the workhorse for analytics / big scans.
Only supports equi-joins (=), because hashing needs equality. No range joins.
Memory: needs a hash table for the build side. If it doesn't fit work_mem/hash grant, it spills to disk (batches) — a classic cause of a "sudden" slowdown. High startup cost (must build before any output), so bad for LIMIT of a few rows.
In the plan: a Hash Join node with a child Hash node feeding it (the build side). Spills show as Batches > 1 / Disk usage.
Merge Join (Sort-Merge)

Mechanic: both inputs must be sorted on the join key; then walk both in lockstep like a zipper, advancing the side with the smaller key. Cost ≈ sort_cost(a) + sort_cost(b) + linear_merge; if inputs are already sorted (e.g., index-ordered scans), the sort cost vanishes and it's very cheap.

Best when: both inputs are large and already sorted on the join key (covering/clustered indexes), or the join also feeds an ORDER BY/merge that wants sorted output. Supports range and equality predicates.
Cost driver: if inputs aren't sorted, the required Sort operators (and their potential disk spill) dominate — that's when the optimizer prefers a hash join instead.
Memory: modest if pre-sorted; the Sorts are the memory/spill risk.
In the plan: Merge Join with (often) Sort or Index Scan children supplying ordered rows.
3. Why the optimizer picks each — the decision

The optimizer is cost-based: it estimates rows via statistics/histograms, computes each candidate operator's cost, and picks the cheapest whole tree. Heuristics that fall out:

Small outer + indexed inner → Nested Loop.
Two big unsorted equi-join inputs, no index → Hash Join.
Two big inputs already sorted on the key (or output needs sorting), or a range join → Merge Join.

The catch: these choices are only as good as the cardinality estimates. A stale/undersampled statistic makes the optimizer think the outer side is 10 rows when it's 10 million → it picks a Nested Loop → the inner index seek runs 10M times → query "hangs." That's why the est-vs-actual comparison is step one of diagnosis.

4. Diagnostic playbook + common mistakes
Capture EXPLAIN (ANALYZE, BUFFERS) (Postgres) / actual plan (SQL Server).
Find the operator with the largest actual time / rows / buffers — the hotspot.
Check est vs actual rows on it and its children. Big skew → update statistics, add/rewrite predicates, avoid WHERE func(col)=... (non-sargable, kills index use), or add multi-column stats.
Match join to data: a Nested Loop with huge loops on a big outer → wanted a Hash/Merge, usually because of a bad estimate or missing index. A Hash/Sort spilling to disk → raise work_mem/memory grant or reduce rows earlier. A Seq/Table Scan on a large table with a selective predicate → missing/unused index.
Only after understanding why, consider a targeted fix (index, updated stats, query rewrite) rather than a blunt optimizer hint.

#### Common mistakes
trusting estimated plans as if they were actuals; assuming "index = always faster" (for a low-selectivity predicate a seq scan + hash join is genuinely cheaper); jamming in join hints to force a merge/hash instead of fixing the underlying estimate; ignoring loops and reading a nested loop's per-execution cost as the total.

Summary — what a strong answer emphasizes

An execution plan is a tree of physical operators with estimated and actual costs; you read it leaves-first and your #1 signal is estimated vs actual rows. The three joins map to data shapes: Nested Loop = small outer + indexed inner (low memory, streaming, OLTP); Hash Join = big unsorted equi-joins with no index (linear, high startup, may spill); Merge Join = big pre-sorted inputs or range joins (cheap if sorted, else Sort cost dominates). The optimizer picks by cost, so most "wrong join" problems trace back to bad cardinality estimates — fix the statistics/predicates, not just the operator.

#### Follow-up
probes: "How would you force/verify a join type?" (Postgres SET enable_nestloop=off to test hypotheses, SQL Server OPTION (HASH JOIN) — for diagnosis, not permanent fixes). "What is a spill and how do you detect it?" (build/sort exceeds memory grant → disk batches; visible as Disk/Batches>1/sort warnings). "Bushy vs left-deep join trees?" and "how do parameter-sniffed plans go wrong?"

**Comparison:**

```
Property	Nested Loop	Hash Join	Merge Join
How it works	For each outer row, probe inner (usually index seek)	Build hash table on smaller input, probe with larger	Sort both inputs, zipper-merge in key order
Join predicate	=, <, >, range, non-equi	equality only	equality and range
Needs index?	Ideally on inner join key	No	Helpful (to avoid the sort)
Input ordering	None	None	Both must be sorted on key
Cost model	outer × inner-probe	build + probe (≈ linear)	sort(a)+sort(b)+merge
Memory	Low (streaming)	Hash table (may spill)	Low if pre-sorted; Sort may spill
Startup cost	Low (streams early — good for LIMIT)	High (build before any output)	High if it must sort first
Best for	Small outer + indexed inner (OLTP)	Big unsorted equi-joins, no index (OLAP)	Big pre-sorted inputs / range joins / needs sorted output
Worst case	Large outer, no inner index → O(m×n)	Build side spills to disk	Unsorted inputs → dominated by Sort cost
```

**Code:**

```sql
-- Postgres: estimated plan only
EXPLAIN
SELECT o.id, c.name
FROM orders o JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'PAID';

-- Postgres: ACTUAL rows/time/spills (runs the query) -- use this to diagnose
EXPLAIN (ANALYZE, BUFFERS)
SELECT o.id, c.name
FROM orders o JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'PAID';

-- Hypothesis testing: disable a join type to see if the optimizer's choice was right
SET enable_nestloop = off;   -- session only; revert after
-- SQL Server equivalent hint (diagnosis, avoid in prod): OPTION (HASH JOIN)

-- Fix the usual root cause: stale statistics driving a bad estimate
ANALYZE orders;                 -- Postgres
-- UPDATE STATISTICS dbo.orders; -- SQL Server
```

**Example input / output:**

```

Reading a Postgres EXPLAIN ANALYZE fragment:

CODE
Copy
Hash Join  (cost=32.5..210.0 rows=1000 width=40)
           (actual time=0.8..12.4 rows=985 loops=1)
  Hash Cond: (o.customer_id = c.id)
  ->  Seq Scan on orders o   (rows=1000)  (actual rows=985)   -- probe side
        Filter: (status = 'PAID')
  ->  Hash  (rows=500)  (actual rows=500)                     -- build side
        ->  Seq Scan on customers c  (actual rows=500)

How to read it:

Root = Hash Join; its Hash child is the build side (customers), the Seq Scan on orders is the probe side.
Compare rows=1000 (est) vs actual rows=985 — close, so estimates are trustworthy here.
loops=1 — no repeated inner execution (contrast a Nested Loop where the inner shows loops=N).
No Batches>1 / Disk note → the hash table fit in memory (no spill). Healthy plan.

Contrast — a problem Nested Loop:

CODE
Copy
Nested Loop (actual rows=2000000 loops=1)
  -> Seq Scan on orders  (est rows=10, actual rows=2000000)   <-- est vs actual blowout
  -> Index Scan on customers  (loops=2000000)                 <-- inner ran 2M times

Diagnosis: bad cardinality estimate on orders (est 10 vs actual 2M) caused a Nested Loop that probes the inner index two million times → should have been a Hash Join. Fix: update statistics / correct the predicate.
```

**Flow:**

1. Small outer + indexed inner
2. Two big unsorted inputsequi-join, no index
3. Both large & already sortedor range join / need order
4. SQL query
5. Cost-based optimizeruses statistics/histograms
6. Data shape?
7. Nested Loopindex seek per outer row
8. Hash Joinbuild then probe
9. Merge Joinsort then zipper-merge
10. Physical plan treeleaves - root
11. Read plan: est vs actual rows,loops, buffers, spills

