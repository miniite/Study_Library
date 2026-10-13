---
type: "practice"
difficulty: "Deep"
---

# How would you paginate a large result set efficiently, and why is OFFSET slow for deep pages?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Two linked things. (1) A practical recipe for paginating a large result set at scale, and (2) a precise, mechanical explanation of why LIMIT ... OFFSET N degrades on deep pages. The interviewer is probing whether I understand what the database actually does when it executes an OFFSET — not just that "it's slow," but the cost model behind it — and whether I know the alternatives and their trade-offs.

#### Assumptions
/ clarifying questions I'd raise:

Is this an API-driven "next page" flow (sequential access) or a "jump to page 4,732" flow (random access)? This changes the answer fundamentally.
Is there a stable, unique, indexed sort key (e.g., created_at, id)? Keyset pagination needs one.
Do we need an exact total count / page numbers, or just infinite scroll?
Read-heavy with concurrent inserts? That affects consistency of pages.

Plan: Explain the mechanics of OFFSET first (the root cause), then walk through the fix hierarchy: keyset/cursor pagination, and when you're forced to keep offset, how to blunt the cost.

Why OFFSET is slow on deep pages (the core mechanics)

The critical misconception is that OFFSET 100000 tells the database to "start reading at row 100,000." It does not. SQL has no notion of a physical row position that satisfies an arbitrary WHERE + ORDER BY. So the database must produce, sort, and then discard the first OFFSET rows before it can return the LIMIT you actually want.

Concretely, for ... ORDER BY created_at DESC LIMIT 20 OFFSET 100000:

The planner scans (via index or table) rows matching the WHERE clause.
It orders them by created_at.
It counts off and throws away the first 100,000 rows — reading each one, applying visibility/WHERE checks, walking the index.
Only rows 100,001–100,020 are returned to you.

So the work is O(OFFSET + LIMIT), not O(LIMIT). Page 1 reads ~20 rows; page 5,000 reads ~100,020 rows to hand you 20. Latency grows linearly with page depth. Deep pages also blow the buffer cache, cause more I/O, and — in Postgres with MVCC — force visibility checks on every skipped tuple. If the sort can't be served fully by an index, you additionally pay a sort of the whole matching set.

A second, subtler problem: OFFSET pagination is not stable under concurrent writes. If a new row is inserted at the top while a user pages, every subsequent page shifts by one, so users see duplicates or skip rows. Offset semantics are "position in a snapshot that no longer exists."

The fix: keyset (cursor) pagination

Instead of "skip N rows," remember the sort-key value of the last row you returned and ask for rows after it. This turns pagination into an indexed range seek — O(LIMIT) regardless of depth.

For a feed sorted by newest first, with a unique tiebreaker id:

SQL
-- Page 1
SELECT id, title, created_at
FROM orders
WHERE tenant_id = $1
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Next page: pass the (created_at, id) of the last row from page 1
SELECT id, title, created_at
FROM orders
WHERE tenant_id = $1
  AND (created_at, id) < ($last_created_at, $last_id)   -- row-value comparison
ORDER BY created_at DESC, id DESC
LIMIT 20;

With a composite index on (tenant_id, created_at DESC, id DESC), the database seeks directly to the cursor position in the B-tree and reads 20 rows. Page 100,000 costs the same as page 1.

Why the tiebreaker matters: created_at alone isn't unique — two rows with the same timestamp would cause skips or repeats at page boundaries. Comparing the composite (created_at, id) gives a total order, so the cursor is unambiguous. Using SQL row-value syntax (created_at, id) < (...) (not created_at < ... OR (created_at = ... AND id < ...)) lets the planner use the index cleanly.

The cursor sent to the client is just the last sort key, usually base64-encoded and opaque: eyJjcmVhdGVkX2F0IjoiMjAyNi0wNy0wNlQxMjozMDowMFoiLCJpZCI6MTIzNDV9.

When you're forced to keep OFFSET

Keyset can't do arbitrary page jumps ("go to page 4,732") or backward random access easily. If the product truly needs numbered pages, mitigate:

Cap the depth. Google/Amazon-style search stops at ~page 100. Most users never go past page 3; deep pages are almost always bots. Capping is the single highest-leverage fix.
Deferred join / "seek then fetch." Do the offset over a narrow, index-only set of primary keys, then join back for the wide columns:
SQL
SELECT o.*
FROM orders o
JOIN (
  SELECT id FROM orders
  WHERE tenant_id = $1
  ORDER BY created_at DESC
  LIMIT 20 OFFSET 100000
) k ON k.id = o.id
ORDER BY o.created_at DESC;
The offset still scans 100k index entries, but they're tiny (index-only), so it's much cheaper than materializing 100k full rows.
Materialized page boundaries / precomputed rank for slowly-changing datasets.
Total counts

SELECT COUNT(*) over a huge filtered set is itself expensive and defeats the purpose. Options: show "1000+" (cap the count), use an estimate (reltuples from pg_class, or EXPLAIN row estimates) for approximate totals, or use infinite scroll and skip counts entirely.

Summary — what a strong answer emphasizes
OFFSET is O(OFFSET+LIMIT): the DB reads and discards all skipped rows because there's no physical "row N" for an arbitrary ordered query. Latency grows linearly with page depth; deep pages also thrash cache and (in MVCC) pay visibility checks. It's also unstable under concurrent inserts.
The right default is keyset/cursor pagination: seek by the last row's sort key using a unique, indexed composite key (sort column + tiebreaker), giving O(LIMIT) at any depth and stable results.
Keyset trades away arbitrary page jumps; if you need those, cap depth, use a deferred (index-only) join, and avoid exact deep counts (estimate or cap).

#### Common mistakes
forgetting the unique tiebreaker (causes duplicates/skips); indexing the sort in the wrong direction so the seek can't be used; using OFFSET for infinite scroll where keyset is strictly better; running COUNT(*) on every page.

#### Follow-up
probes to expect: "How do you page backwards?" (reverse the comparison + ORDER BY, then reverse the result set in the app). "How do you keep results stable across writes?" (keyset is naturally stable; or snapshot with a created_at <= request_time upper bound).

**Comparison:**

```
Aspect	OFFSET pagination	Keyset (cursor) pagination
Deep-page cost	O(OFFSET + LIMIT) — grows with depth	O(LIMIT) — constant
Random "jump to page K"	Yes	No (sequential only)
Stable under concurrent inserts	No (drift → dup/skip)	Yes
Needs unique indexed sort key	No	Yes (sort col + tiebreaker)
Exact page numbers / total count	Natural	Awkward (estimate or omit)
Best for	Small sets, admin tables, "page N" UIs	Feeds, infinite scroll, APIs, large tables
```

**Code:**

```sql
-- Keyset pagination with a composite unique sort key + supporting index
CREATE INDEX idx_orders_feed
  ON orders (tenant_id, created_at DESC, id DESC);

-- Page N (cursor = last row's (created_at, id) from previous page)
SELECT id, title, created_at
FROM orders
WHERE tenant_id = $1
  AND (created_at, id) < ($last_created_at, $last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Deferred join to blunt a required OFFSET (index-only skip, then fetch wide cols)
SELECT o.*
FROM orders o
JOIN (
  SELECT id
  FROM orders
  WHERE tenant_id = $1
  ORDER BY created_at DESC
  LIMIT 20 OFFSET 100000
) k ON k.id = o.id
ORDER BY o.created_at DESC;
```

**Example input / output:**

```

Trace of the cost difference (20 rows/page):

CODE
Copy
Query: ORDER BY created_at DESC LIMIT 20 OFFSET N

N = 0        -> DB reads ~20 rows           -> ~1 ms
N = 20       -> DB reads ~40 rows           -> ~1 ms
N = 100000   -> DB reads ~100020 rows,      -> ~800 ms   (99.98% discarded)
                returns last 20

Keyset:  WHERE (created_at,id) < (cursor) LIMIT 20
any page -> B-tree seek + read ~20 rows     -> ~1 ms   (constant)
```

**Data:**

```json
{
  "data": [ { "id": 88231, "title": "Order #88231", "created_at": "2026-07-06T12:30:00Z" } ],
  "page_info": {
    "has_next_page": true,
    "end_cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNi0wNy0wNlQxMjozMDowMFoiLCJpZCI6ODgyMzF9"
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Need pagination
6. Random page jumps required?
7. Keyset/cursor: seek by last sortkeyO LIMIT, stable
8. Dataset large / deep pageshit?
9. Plain OFFSET is fine
10. Cap max depth
11. Deferred index-only join forthe offset
12. Estimate/omit total count

