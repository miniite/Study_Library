---
type: "practice"
difficulty: "Intermediate"
---

# What is database indexing and how does it speed up queries?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pagination Strategies](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Pagination_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: "What is a database index, and mechanically why does it make queries faster?" This is a bread-and-butter backend/system-design question. At the intermediate level, a good answer isn't just "an index is like a book's index" — the interviewer wants to see that you understand (1) the data structure underneath (usually a B+ tree), (2) which query patterns it accelerates and which it can't, (3) the cost model (page reads / disk I/O), and (4) the trade-offs so you don't over-index.

What the interviewer is probing: Do you know what happens at the storage-engine level? Can you reason about WHERE, ORDER BY, JOIN, range scans, covering indexes, and selectivity? Can you read an EXPLAIN plan and tell a seek from a scan?

#### Assumptions
/ clarifying questions I'd state: I'll assume a classic relational engine (Postgres/MySQL InnoDB) using B+ tree indexes on disk-backed pages. I'd clarify: read-heavy vs write-heavy? Point lookups or range queries? Cardinality of the column? Those change the answer.

Plan: Define what an index is → explain the B+ tree structure and why it turns O(n) into O(log n) → walk a concrete example with page counts → cover the index types and access patterns it helps → note selectivity and covering indexes → summarize.

What an index actually is

A table by default is stored as a heap (or as a clustered structure keyed on the primary key, in InnoDB). Without any index, answering SELECT * FROM users WHERE email = 'x@y.com' requires a full table scan: the engine reads every row (every data page) and checks the predicate. For a 10-million-row table that might be hundreds of thousands of 8 KB pages — millions of comparisons and a lot of disk I/O.

An index is a separate, sorted, auxiliary data structure that maps column value → location of the row (a row pointer / primary-key value). It's redundant data maintained by the database specifically to make lookups fast, at the cost of extra storage and slower writes.

The data structure: a B+ tree

Almost all general-purpose indexes are B+ trees (balanced, high-fan-out trees tuned for disk). Key properties:

Every leaf is at the same depth, so lookups are worst-case balanced.
Each node is a disk page (e.g., 8–16 KB) holding hundreds of keys, so the tree is very wide and shallow — fan-out of a few hundred means a billion rows fit in ~4–5 levels.
Leaf nodes are linked in sorted order (a doubly linked list). This is the reason a B+ tree accelerates not just equality but range scans and ORDER BY.

Why it speeds things up — the math. Lookup cost is proportional to the tree height, which is O(log_f N) where f is the fan-out. With fan-out ~400 and 100 M rows, height ≈ log_400(100,000,000) ≈ 3. So a point lookup touches ~3–4 pages instead of scanning ~1,000,000+ pages. Because the top levels are almost always cached in RAM, a lookup is often just one or two physical disk reads. That's the whole game: indexes convert a linear scan (O(n) page reads) into a logarithmic tree descent (O(log n) page reads).

A concrete walk-through

Table orders(id PK, user_id, created_at, status, amount), 50 M rows, ~500,000 data pages.

Query: SELECT * FROM orders WHERE user_id = 42; (say 40 rows match).

No index: scan all ~500,000 pages → ~500,000 reads. Slow.
With INDEX(user_id): descend the B+ tree (≈4 page reads) to the first leaf entry for user_id=42, walk the linked leaves collecting the 40 matching row pointers, then fetch those 40 rows. Roughly 4 + 40 reads instead of 500,000 — three to four orders of magnitude fewer I/Os.
What access patterns an index helps (and doesn't)
Equality / point lookup (WHERE user_id = 42): a seek to the leaf. Fast.
Range (WHERE created_at BETWEEN a AND b): seek to the start, then walk sorted leaves. Fast because leaves are sorted and linked.
ORDER BY / GROUP BY on the indexed column: the data is already sorted, so the engine can skip a sort step.
Prefix of a composite index: INDEX(a, b, c) helps WHERE a=…, WHERE a=… AND b=…, but not WHERE b=… alone — this is the leftmost-prefix rule.
Joins: an index on the join column lets the engine do an index-nested-loop join instead of a hash/scan.

An index does not help when: the predicate can't use it (WHERE UPPER(email)=… unless there's a functional index; LIKE '%foo' with a leading wildcard; WHERE b=… on INDEX(a,b)), or when the query returns a large fraction of the table — then a full scan is actually cheaper than millions of random-access "bookmark lookups."

Selectivity and the "tipping point"

The optimizer is cost-based. An index on a low-selectivity column (e.g., status with 3 values) is often ignored, because fetching, say, 40% of the rows via random I/O is slower than a sequential scan. Indexes pay off on high-selectivity / high-cardinality columns (email, user_id, order_id) where a lookup returns a tiny slice. Rule of thumb: an index is useful when it filters to well under ~5–10% of rows.

Covering indexes and clustered indexes
Covering index: if the index itself contains every column the query needs (in the key or as INCLUDE columns), the engine answers entirely from the index and never touches the base table — no bookmark lookups at all. Hugely faster for hot queries. Example: INDEX(user_id) INCLUDE (status) fully serves SELECT status FROM orders WHERE user_id=42.
Clustered index (InnoDB primary key): the table rows are physically stored in PK order inside the B+ tree leaves. So a PK lookup returns the whole row directly; secondary indexes store the PK as the pointer, meaning a secondary lookup does one extra hop into the clustered index.
Summary — what a strong answer emphasizes

An index is a sorted, B+ tree auxiliary structure that maps values to row locations, turning an O(n) full scan into an O(log n) tree descent — often just a couple of disk reads for a lookup on a billion-row table. It accelerates equality, ranges, sorting, and joins because leaves are sorted and linked. But it only helps selective predicates in a sargable form, it's constrained by the leftmost-prefix rule, and covering indexes are the fastest of all because they avoid the base-table fetch. A strong candidate finishes with the trade-off: indexes cost storage and slow down writes (every insert/update/delete must maintain them), so you index deliberately based on real query patterns, not everywhere.

#### Common mistakes
to avoid: saying "indexes always make queries faster" (false for low selectivity / write-heavy tables); forgetting the leftmost-prefix rule; not knowing that leading-wildcard LIKE and wrapping a column in a function defeat the index; confusing a clustered PK with a secondary index.

Likely follow-ups: "What's the difference between a clustered and non-clustered index?", "Why would the optimizer ignore your index?", "Design the index(es) for this specific query" — read the EXPLAIN and reason about selectivity and column order.

**Comparison:**

```
Query pattern	Index that helps	Why
WHERE user_id = 42 (equality)	INDEX(user_id)	B+ tree seek to a leaf
WHERE created_at BETWEEN a AND b (range)	INDEX(created_at)	Seek + walk sorted, linked leaves
ORDER BY created_at	INDEX(created_at)	Data already sorted, skip sort
WHERE user_id=? AND created_at=?	INDEX(user_id, created_at)	Composite, leftmost-prefix satisfied
WHERE created_at=? only	INDEX(user_id, created_at) does not help	Violates leftmost-prefix rule
WHERE email LIKE '%foo'	none	Leading wildcard is non-sargable
WHERE status='OPEN' (3 values)	often ignored	Low selectivity → scan cheaper
Index concept	What it means	Speed effect
Point lookup	Equality seek	~log(N) page reads
Covering index	Index holds all needed columns	Zero base-table fetches (fastest)
Clustered index (InnoDB PK)	Rows stored physically in key order	PK lookup returns full row directly
Secondary index	Stores PK as pointer	One extra hop into clustered index
```

**Code:**

```sql
-- A simple secondary index (B+ tree by default)
CREATE INDEX idx_orders_user ON orders (user_id);

-- Composite index: helps WHERE user_id=? [AND created_at range/order]
-- Leftmost-prefix rule: does NOT help a lone WHERE created_at=?
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);

-- Covering index (Postgres): query served entirely from the index
CREATE INDEX idx_orders_user_cov ON orders (user_id) INCLUDE (status, amount);

-- Inspect whether an index is actually used (seek vs scan)
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 42;
```

**Example input / output:**

```text
Dry-run of `SELECT * FROM orders WHERE user_id = 42` on a 50M-row table:

WITHOUT index:
  Seq Scan on orders  -> reads ~500,000 pages, checks 50,000,000 rows -> matches 40. Slow.

WITH INDEX(user_id):
  Index Scan using idx_orders_user
    1. Descend B+ tree root->internal->leaf   (~4 page reads, mostly cached)
    2. Walk linked leaf entries for user_id=42 (40 pointers)
    3. Fetch 40 rows from heap/clustered index (~40 reads)
  Total ~44 reads instead of ~500,000.

EXPLAIN output shape:
  -> "Index Scan using idx_orders_user"  = index seek (good)
  -> "Seq Scan on orders"                = full scan (index unused / not selective)
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. SELECT * FROM orders WHEREuser_id = 42
6. Index on user_id?
7. Seq Scan: read ~500k pages,check 50M rows
8. B+ tree root
9. internal node
10. leaf (sorted, linked)
11. Walk linked leaves foruser_id=42 -> 40 row pointers
12. Covering index?
13. Answer from index only (nobase fetch)
14. Fetch 40 rows fromheap/clustered index

