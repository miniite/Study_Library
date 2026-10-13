---
type: "practice"
difficulty: "Deep"
---

# How does a database index change the complexity of a lookup query from O(n) to O(log n)?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Connect the practical world (adding a DB index) to the theory (asymptotic complexity). Specifically: why does an unindexed WHERE col = ? lookup cost O(n), and why does an index bring it down to O(log n)? This is a "deep understanding" backend/fullstack question at strong product companies, so I need to explain the data structure underneath the index (a B-tree / B+tree), not just say "indexes make queries fast."

What they're probing for: Do I know that a heap table has no order, so the DB must scan every row (a full table scan) → O(n)? Do I know an index is a sorted, balanced tree that lets the engine do the equivalent of binary search → O(log n)? Can I state the constant factors that matter in practice (tree fanout, disk I/O, the extra lookup back to the heap)? And do I know the limits — when an index does NOT give O(log n) (leading-wildcard LIKE, functions on the column, low selectivity, range vs equality)?

#### Assumptions
/ clarifying questions: I'll assume a relational DB (Postgres/MySQL-InnoDB) and a B-tree index (the default). I'd clarify: equality lookup or range? Is it a covering query or does it need extra columns from the heap? Is the column highly selective (many distinct values)?

Plan: describe the unindexed scan → derive O(n) → introduce the B-tree index → derive O(log n) via tree height → discuss the real constant factors and the base of the log → note when it does NOT hold. I'll add a worked example with numbers.

Without an index: O(n) full table scan

A table is stored as an unordered heap (or a clustered pile of pages). There is no structure that says "row with id = 42 lives here." So to answer SELECT * FROM users WHERE email = 'x', the database has no choice but to read every row and test the predicate. If there are n rows, that is n comparisons and n rows' worth of page reads. Time grows linearly with table size → O(n). Double the rows, double the work. At 10 million rows this is millions of comparisons and lots of disk I/O per lookup — unusable for an OLTP endpoint.

With an index: O(log n) because it's a balanced sorted tree

A B-tree (Postgres/MySQL default) or B+tree index keeps a sorted copy of the indexed column plus a pointer (row id / primary key) back to the actual row. Because the data is sorted and the tree is height-balanced, the engine finds a value the same way binary search finds a value in a sorted array: start at the root, compare, descend into the one child subtree that can contain the key, and repeat.

Each step eliminates a fraction of the remaining rows — that halving-like reduction is exactly what makes the cost logarithmic. The number of steps equals the height of the tree. For a binary tree the height is log₂(n); for a B-tree with fanout b (hundreds of keys per node) the height is log_b(n), which is even shallower. Either way it's O(log n) — changing the base of the log is just a constant factor, so asymptotically all are O(log n).

So the lookup goes from touching n rows to touching only ~height nodes. That's the whole story: an index replaces a linear scan of unordered data with a logarithmic descent of a sorted, balanced tree.

Why the constant factors are actually huge (the practical punchline)

The asymptotic difference is O(n) → O(log n), but the practical speedup is even more dramatic than that because of fanout and disk I/O:

B-tree nodes are page-sized and hold hundreds of keys, so fanout b ≈ 100–500. A tree over 1 billion rows is only ~3–4 levels deep (log₄₀₀(10⁹) ≈ 3.3). That means ~4 page reads to find any row, vs. reading the entire billion-row table.
Databases cache upper index levels in RAM, so in practice a point lookup is often 1–2 physical reads.

So O(log n) isn't just "a bit better" — for a billion rows it's ~4 steps vs. a billion.

The subtlety: the second lookup (index → heap)

An index entry stores the key + a pointer, not the full row. After the O(log n) descent finds the pointer, the engine often does one more read to fetch the actual row from the heap (a "heap fetch" / bookmark lookup). That's still O(log n) overall (one extra O(1)-ish read). A covering index (one that already contains every column the query needs, e.g. via INCLUDE in Postgres or a composite index) skips the heap fetch entirely — index-only scan.

When an index does NOT give you O(log n) (the mistakes they probe)

An index only helps if the query can exploit the sort order:

Leading-wildcard LIKE '%foo' — can't seek into a sorted tree because the prefix is unknown → full scan, O(n).
Function/expression on the column — WHERE lower(email) = 'x' can't use an index on email (fix: index the expression lower(email)).
Low selectivity — indexing a boolean/gender column returns a huge fraction of rows; the planner rationally chooses a seq scan because random I/O per row is worse than a sequential scan.
Range queries — an index still helps (seek to the start, then scan the range), giving O(log n + k) for k matches — not pure O(log n).
Type mismatch / implicit cast on the predicate can silently disable the index.
Summary — what a strong answer emphasizes

Unindexed lookup is O(n) because a heap is unordered, forcing a full table scan. An index is a balanced, sorted B-tree, so a lookup becomes a binary-search-style descent whose cost equals the tree height, O(log_b n) = O(log n) — each level eliminates most remaining rows. In practice the high fanout makes even billion-row tables only 3–4 levels deep, so the real win dwarfs the asymptotic label. Mention the heap fetch (and how covering indexes remove it) and the cases where the index does not apply (leading wildcards, functions on the column, low selectivity). That combination — data structure + complexity + real constants + limits — is what "deep understanding" means here.

#### Follow-up
probes: "Why B-tree and not a hash index?" → hash gives O(1) equality but no ordering, so no range/sort support; B-tree gives O(log n) and range/ORDER BY. "What does the index cost you?" → slower writes and extra storage (see the read-vs-write trade-off). "Composite index column order?" → leftmost-prefix rule.

**Comparison:**

```
Aspect	No index (heap / seq scan)	B-tree index
Data order	Unordered heap	Sorted + balanced tree
Lookup mechanism	Test every row	Binary-search-style descent
Point-lookup complexity	O(n)	O(log n)
Rows touched (1e9 rows)	~1,000,000,000	~4 (tree height)
Range query	O(n)	O(log n + k)
Extra heap fetch	n/a	1 read (0 if covering index)
When it fails	—	leading %LIKE, func(col), low selectivity, type mismatch
Hash index (contrast)	—	O(1) equality but no ranges/ordering
```

**Code:**

```sql
-- Unindexed: planner must scan every row -> O(n)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@b.com';
-- Seq Scan on users  (cost=0.00..18334.00 rows=1 width=...)   <- reads all rows

-- Create a B-tree index (default type)
CREATE INDEX idx_users_email ON users (email);

-- Now: index seek -> O(log n) descent + heap fetch
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@b.com';
-- Index Scan using idx_users_email  (cost=0.42..8.44 rows=1)  <- ~tree height reads

-- Covering / index-only scan: no heap fetch (Postgres INCLUDE)
CREATE INDEX idx_users_email_cov ON users (email) INCLUDE (name, created_at);
SELECT name, created_at FROM users WHERE email = 'a@b.com';  -- Index Only Scan

-- Index that does NOT help (function on column) -> falls back to O(n)
SELECT * FROM users WHERE lower(email) = 'a@b.com';          -- can't use idx_users_email
-- Fix: index the expression
CREATE INDEX idx_users_email_lower ON users (lower(email));
```

**Example input / output:**

```text
Table users, n = 1,000,000,000 rows, B-tree fanout b ~= 400.

WITHOUT index (Seq Scan):
  work           = ~n row comparisons
  complexity     = O(n)
  ~ 1,000,000,000 comparisons  (reads whole table)

WITH B-tree index (Index Scan):
  steps          = tree height = ceil(log_400(1e9)) ~= 4
  complexity     = O(log n)
  ~ 4 node reads  +  1 heap fetch  ~= 5 page reads

Speedup factor ~= 1e9 / 5  ~= 200,000,000x fewer reads for a point lookup.

Range query WHERE age BETWEEN 30 AND 40 returning k rows:
  seek to start = O(log n), then scan k matches
  complexity     = O(log n + k)   (not pure O(log n))
```

**Flow:**

1. No index / func on col /leading %
2. B-tree index present
3. Yes
4. No
5. WHERE email = 'x'
6. Usable index?
7. Seq Scan: test all n rowsO(n)
8. Root node
9. compare key, descend 1 child(eliminates most rows)
10. intermediate levels
11. Leaf: key + row pointer~log_b(n) steps = O(log n)
12. Covering index?
13. Index-only scan, done
14. +1 heap fetch for full row

