---
type: "practice"
difficulty: "Deep"
---

# What is the difference between offset-based and cursor-based pagination?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pagination Strategies](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Pagination_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

"What is the difference between offset-based and cursor-based pagination?" is a deep-understanding probe. The interviewer isn't looking for a dictionary definition; they want you to explain how each mechanically works against the database, why one degrades at scale, why one duplicates rows under concurrent writes, and when to pick each. Companies like Meta, Google, and Amazon ask this because their feeds and catalogs live and die on deep-page performance and result stability. A senior answer connects the SQL, the index behavior, the O-complexity, the correctness under mutation, and the API contract.

#### Assumptions
I'll state: a relational store with a sortable result set and a unique primary key; both approaches need a deterministic ORDER BY. I'll use (created_at, id) as the running example ordering.

Plan: define each mechanism, contrast on the four axes that actually matter (performance, stability, navigability, total-count), show the SQL and a trace of the duplicate bug, then give a decision rule and edge cases.

Offset-based pagination — how it works

You tell the database "produce the ordered result, then skip the first N rows and return the next K":

ORDER BY created_at DESC, id DESC LIMIT 20 OFFSET 100

The client thinks in page numbers: offset = (page - 1) * pageSize. Page 6 of 20 → OFFSET 100.

The crucial mechanical fact: OFFSET is positional, not indexed-anchored. Even with a perfect index on the sort columns, the engine must walk and discard all OFFSET rows before it can emit the ones you want. The cost is O(offset + limit) — reading page 1 is cheap, but reading page 50,000 forces the engine to traverse a million index/heap entries just to throw them away. This is the deep-page latency cliff.

Cursor-based (keyset / seek) pagination — how it works

Instead of a numeric skip, the client passes back a cursor: the sort-key values of the last row it saw. The next query is a range seek:

WHERE (created_at, id) < (:lastCreatedAt, :lastId) ORDER BY created_at DESC, id DESC LIMIT 20

With a composite index on (created_at, id), the database binary-searches straight to that position in the index and reads exactly the next 20 rows. Cost is O(limit) — page 50,000 is as fast as page 1 because there's no skipping; the WHERE clause jumps to the anchor.

The cursor is normally opaque — a base64/encoded token — so clients treat it as a token they hand back, letting the server change the internal encoding freely.

The four differences that matter

1. Performance at depth. Offset is O(offset): deep pages get linearly slower and burn CPU/IO on discarded rows. Cursor is O(limit): constant cost at any depth. This alone drives most large-scale decisions.

2. Stability under concurrent writes. Offset positions are relative to the current result set. If rows are inserted/deleted above your current page between requests, everything shifts: you either see a row twice (duplicate) or skip one entirely. Cursor pagination anchors to a real row's key value, so inserts/deletes elsewhere don't move your window — no duplicates, no skips. For live feeds (Meta), chat, and activity logs, this correctness property is decisive.

3. Navigability. Offset gives random access: "jump to page 500," "last page," and an easy total-page count. Cursor is sequential only — you can go next (and, with a symmetric cursor, previous), but you can't jump to an arbitrary page number, because there's no notion of "the 500th page" without counting.

4. Total counts. Offset pairs naturally with COUNT(*) to render "Page 3 of 61." Cursor usually omits exact totals (they're expensive on large sets) in favor of a hasMore boolean and "load more."

The tiebreaker — the detail that proves depth

Cursor pagination requires a unique, deterministic tiebreaker in the ORDER BY. If you paginate by created_at alone and two rows share a timestamp, a plain created_at < :last can skip or duplicate the tied rows at the page boundary. Adding the unique id ((created_at, id)) makes the ordering total, so the seek is exact. Mentioning this unprompted signals you've actually shipped keyset pagination. Offset technically also needs a stable ORDER BY (otherwise page contents are nondeterministic), but its brokenness under writes exists regardless.

API contract difference
Offset: GET /items?page=3&pageSize=20 → { data, page, pageSize, totalItems, totalPages }.
Cursor: GET /items?cursor=<opaque>&limit=20 → { data, nextCursor, hasMore }.

Both should cap page size (e.g. max 100). Cursors are also more cache-friendly (a token maps to a stable slice) and more secure against "give me offset 9,999,999" DoS-style deep scans.

Search engines and distributed stores

The same split appears beyond SQL. Elasticsearch caps deep from/size (default 10k window) because deep offset is costly across shards, and provides search_after — a cursor — for deep paging, plus scroll/PIT for full exports. This reinforces the point: at scale, the industry standardizes on cursor semantics.

Decision rule
Choose offset when the dataset is small-to-moderate and fairly static, and the product genuinely needs numbered pages, jump-to-page, or an exact total (admin tables, reports, search-result pages with page numbers).
Choose cursor when the dataset is large or actively mutating, or the UX is infinite scroll / "load more" (social feeds, timelines, chat, notifications, activity logs, large exports).

#### Common mistakes
/ follow-up probes
Claiming "cursor is always better" — it can't do jump-to-page or cheap totals.
Forgetting the unique tiebreaker → subtle duplicate/skip bugs at timestamp ties.
Using offset on tens of millions of rows and being surprised by deep-page latency.
Exposing a raw (non-opaque) cursor, coupling clients to internal columns.
No page-size cap. Follow-ups often ask: "How do you do previous?" (symmetric reversed cursor), "How do you show a total with cursor?" (approximate/cached count), and "What breaks on a lagging read replica?" (read-your-writes — a just-created row may be missing).
Summary

Offset skips N rows and returns the next K — simple, supports page numbers and totals, but is O(offset) slow on deep pages and unstable under concurrent writes (duplicates/skips). Cursor seeks after a remembered, indexed (sortKey, id) position — O(limit) fast at any depth and write-stable, at the cost of no random access and no cheap totals. Pick offset for small/static, numbered-page UIs; pick cursor for large/mutating data and infinite scroll. The senior signal is naming the unique tiebreaker and the O-complexity difference.

**Comparison:**

```
Dimension	Offset-based	Cursor-based (keyset)
Query form	LIMIT k OFFSET n	WHERE (key) < :cursor LIMIT k
Deep-page cost	O(offset + k) — degrades	O(k) — constant
Stable under insert/delete	No (duplicates / skips)	Yes (anchored to a row key)
Jump to arbitrary page	Yes	No (sequential)
Previous page	Trivial	Needs symmetric reversed cursor
Exact total / page count	Easy (COUNT)	Costly — usually hasMore only
Index need	Sort cols indexed	Composite (sortKey, id) mandatory
Cursor exposed to client	page/pageSize ints	opaque token
Ideal use	Small/static, numbered pages, reports	Large/mutating, feeds, infinite scroll
```

**Code:**

```sql
-- OFFSET: engine walks and DISCARDS 100 rows, then returns 20  -> O(offset)
SELECT id, body, created_at
FROM posts
WHERE feed_id = 7
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 100;                 -- page 6, 20 per page

-- CURSOR/KEYSET: seek straight past the last-seen row  -> O(limit)
SELECT id, body, created_at
FROM posts
WHERE feed_id = 7
  AND (created_at, id) < (:lastCreatedAt, :lastId)   -- unique id = tiebreaker
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Index that makes the cursor seek fast and the ordering stable
CREATE INDEX idx_posts_feed_created_id
  ON posts (feed_id, created_at DESC, id DESC);
```

**Example input / output:**

```

Trace of the offset DUPLICATE bug vs cursor stability (page size 3):

CODE
Copy
Ordered feed (newest first): P1 P2 P3 P4 P5 P6 ...

t0  OFFSET page 1: returns P1 P2 P3
    (cursor version: returns P1 P2 P3, nextCursor -> key(P3))

t1  A new post P0 is inserted at the TOP (concurrent write).
    Feed is now: P0 P1 P2 P3 P4 P5 P6 ...

t2  OFFSET page 2 (OFFSET 3): rows 4,5,6 of the NEW list = P3 P4 P5
        -> P3 is returned AGAIN  ==> DUPLICATE (P0 also pushed the window)

t2  CURSOR page 2 (WHERE key < key(P3)): returns P4 P5 P6
        -> no duplicate, no skip; insertion of P0 did not disturb the window

Deep-page cost illustration:

CODE
Copy
OFFSET 1,000,000 LIMIT 20  -> engine reads ~1,000,020 rows, discards 1,000,000  (slow)
CURSOR at same depth       -> index seek + read 20 rows                          (fast)
```

**Data:**

```json
// Offset response — page numbers + total
{ "data": [ /* 20 posts */ ], "page": 6, "pageSize": 20,
  "totalItems": 12480, "totalPages": 624 }

// Cursor response — token + hasMore, no total
{ "data": [ /* 20 posts */ ],
  "nextCursor": "WyIyMDI2LTA2LTMwVDA5OjU4OjAwWiIsOTEwMV0",
  "hasMore": true }
```

**Flow:**

1. Cursor
2. cursor=key(lastRow), limit=20
3. WHERE (created_at,id) < key
4. index seek to anchor
5. return next 20 -> O(limit)
6. Offset
7. page=6, size=20
8. OFFSET 100 LIMIT 20
9. walk + discard 100 rows
10. return 20 -> O(offset)

