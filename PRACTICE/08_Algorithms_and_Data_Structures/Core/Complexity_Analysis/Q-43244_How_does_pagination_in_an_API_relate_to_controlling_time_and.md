---
type: "practice"
difficulty: "Intermediate"
---

# How does pagination in an API relate to controlling time and space complexity of a request?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: How does the design decision to paginate an API endpoint (return results in small pages instead of one giant response) act as a lever to bound the time (CPU/IO the server spends per request) and space (memory the server, network, and client hold) that a single request consumes?

What the interviewer is probing: Whether you connect an API design pattern to the underlying complexity of the work. A strong candidate ties pagination directly to: bounding result-set size from O(N) to O(page_size), bounding memory footprint, protecting the database and downstream, and understanding the trade-offs between offset-based and cursor/keyset-based pagination — because the wrong pagination scheme can silently reintroduce the O(N) cost you tried to remove.

Clarifying questions I'd raise:

Is the dataset large and growing (feed, search results, transaction history)? Pagination matters most there.
Do consumers need random page access ("jump to page 500") or just sequential scroll ("load more")? This decides offset vs cursor.
Can the data change under the reader (inserts/deletes) mid-scroll? Affects consistency of the scheme.

Plan: State what unbounded responses cost → show how pagination caps time and space per request → contrast offset vs keyset complexity (the crux) → cover the whole-scan cost and total-work view → summarize.

First principles: what an unbounded response costs

Suppose a table has N rows matching a query. If the API returns all of them in one response, a single request costs:

Server time: O(N) to fetch, serialize (JSON encode), and send — plus the DB's O(N) scan/sort.
Server memory: O(N) to buffer the result set and the serialized payload in RAM.
Network: O(N) bytes transferred; O(N) latency for the last byte.
Client: O(N) memory to parse and hold, and O(N) render time.

As N grows (a user with 2M transactions, a viral post with 10M comments), this becomes slow, memory-hungry, and a denial-of-service risk — one request can exhaust a worker's heap or saturate the DB. The response time is also unbounded and unpredictable.

Pagination replaces N with a bounded page size k in the per-request cost. Instead of one O(N) request, you serve ⌈N/k⌉ requests, each doing O(k) work. The client fetches only what it needs, when it needs it.

How pagination bounds time and space per request

With a fixed page size k (say 20–100), each request:

Time: O(k) to serialize and send the page (the DB fetch time depends on the scheme, see below).
Server memory: O(k) — you buffer only one page, not the whole table. This is the key space win: worker memory stays flat regardless of N.
Network / client: O(k) bytes and O(k) parse/render per request.

So pagination turns per-request complexity from O(N) into O(k) (a constant, since k is capped). That gives predictable latency, flat memory, and a natural back-pressure/rate-limit boundary — a client can't ask for the whole universe in one shot. You also enforce a max page size server-side so a caller can't pass limit=1000000 and defeat the whole point.

The crux: offset vs cursor (keyset) pagination

Where the O(k) claim holds depends on how you locate the page. This is what separates a junior answer from a senior one.

Offset / LIMIT-OFFSET pagination

SELECT ... ORDER BY id LIMIT k OFFSET m. To return page p, the database must scan and discard m = p·k rows to reach the start of the page, then read k. So fetching a deep page is O(m + k) = O(p·k) time — it degrades linearly with page depth. Page 1 is fast; page 10,000 rescans 200,000 rows every time. Memory per request is still O(k), but time is not bounded — deep pages get progressively slower and hammer the DB. Offset pagination also suffers drift: if rows are inserted/deleted while paging, you can skip or duplicate rows.

Cursor / keyset ("seek") pagination

Instead of an offset, you pass the last item's sort key as a cursor: WHERE (created_at, id) < (:last_ts, :last_id) ORDER BY created_at, id LIMIT k. With an index on the sort key, the DB seeks directly to the position and reads k rows — O(log N + k) (index seek) or effectively O(k) per page regardless of depth. This is the scheme that actually delivers the bounded per-request complexity pagination promises, and it's stable under concurrent inserts (no drift for the anchor key). The trade-off: no random "jump to page 42" — it's inherently sequential ("next"/"prev").

So: pagination bounds space to O(k) in every scheme, but only keyset/cursor pagination also bounds time to O(k) at any depth. Offset pagination silently reintroduces O(depth) time. Naming this is the mark of a strong answer.

The total-work view (don't miss it)

Pagination bounds per-request cost, but scanning the entire dataset across all pages is still Θ(N) total work — you've spread it over many requests, you haven't made it free. This is usually the point: work is done lazily and only for data actually consumed. If a client abandons after page 1, you never paid for the rest. Pagination is fundamentally about amortizing / deferring the O(N) cost and capping the peak resource use of any single request.

Related bounding techniques pagination enables
Streaming / chunked transfer is the same idea for a single logical response — bound memory to a window instead of buffering everything.
Content-Range / total counts: computing an exact total is itself O(N); cursor APIs often omit it or estimate it precisely to avoid that cost.
Rate limiting + max page size together cap the total work any client can pull per unit time.
Summary — what a strong answer emphasizes

Pagination is a complexity-bounding contract: by returning at most k items per request, it caps per-request space at O(k) (flat server/client memory, flat network payload) and, with keyset/cursor pagination and a proper index, caps per-request time at O(k) at any depth. Offset pagination bounds memory but not time — deep pages cost O(offset). The full scan is still Θ(N) total, spread across requests and paid only for data actually consumed. This protects the database, keeps worker memory flat, makes latency predictable, and prevents a single request from becoming a DoS.

#### Common mistakes
claiming offset pagination is O(k) at all depths (it's O(offset+k)); forgetting the server must enforce a max limit; ignoring drift/consistency under concurrent writes; conflating per-request cost with total cost.

#### Follow-up
probes: "Why is OFFSET 100000 slow?" (scans and discards 100k rows); "How do you paginate a feed that's constantly changing?" (keyset cursor on a stable sort key); "How do you give a total count cheaply?" (approximate/estimated count, or omit it).

**Comparison:**

```
Aspect	No pagination (return all N)	Offset pagination	Keyset / cursor pagination
Per-request server memory	O(N)	O(k)	O(k)
Per-request time	O(N)	O(offset + k) — degrades with depth	O(log N + k) ≈ O(k) at any depth
Network payload / request	O(N)	O(k)	O(k)
Consistency under writes	n/a	drifts (skip/dup rows)	stable on anchor key
Random "jump to page X"	n/a	yes	no (sequential next/prev)
Total work for full scan	O(N)	Θ(N)	Θ(N)
```

**Code:**

```

-- OFFSET pagination: memory O(k), but TIME O(offset + k) -> slow on deep pages SELECT id, title, created_at FROM posts WHERE author_id = :uid ORDER BY created_at DESC, id DESC LIMIT 20 OFFSET 100000; -- DB scans+discards 100000 rows first

-- KEYSET / CURSOR pagination: memory O(k) AND time O(log N + k) at any depth -- cursor = (last_created_at, last_id) from the previous page SELECT id, title, created_at FROM posts WHERE author_id = :uid AND (created_at, id) < (:last_created_at, :last_id) -- seek, not scan ORDER BY created_at DESC, id DESC LIMIT 20; -- requires index: (author_id, created_at DESC, id DESC)

CODE
Copy
```javascript
// API layer: enforce a max page size so a caller can't defeat pagination
function parsePaging(query) {
  const MAX = 100, DEFAULT = 20;
  const limit = Math.min(parseInt(query.limit) || DEFAULT, MAX); // cap -> O(k)
  const cursor = query.cursor ? decodeCursor(query.cursor) : null;
  return { limit, cursor };
}
```

**Example input / output:**

```text
Request  (page 1):  GET /v1/posts?author=42&limit=20
Response:           20 items + next_cursor=C1, has_more=true
                    server work: fetch 20 rows, buffer 20 rows  -> time O(k), mem O(k)

Request  (page 2):  GET /v1/posts?author=42&limit=20&cursor=C1
Response:           next 20 items + next_cursor=C2
                    keyset: index-seek to C1, read 20        -> O(log N + k)   (still fast)
                    offset OFFSET 20 : scan+discard 20, read 20 -> cheap here...

Deep page (page 5001):
   OFFSET 100000 : DB scans+discards 100,000 rows, then reads 20 -> O(100020)  SLOW
   cursor C5000  : index-seek straight to anchor, read 20        -> O(log N + 20)  FAST

Total to read all N=100,020 rows: Theta(N) either way, but split into ~5001 O(k) requests.
```

**Data:**

```json
// Cursor-paginated response envelope: bounded page + opaque next cursor
{
  "data": [
    { "id": 8842, "title": "Ship v2", "created_at": "2026-07-01T10:00:00Z" },
    { "id": 8830, "title": "Fix auth", "created_at": "2026-07-01T09:12:00Z" }
  ],
  "page": { "size": 20, "returned": 20 },
  "next_cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNi0wNy0wMVQwOToxMjowMFoiLCJpZCI6ODgzMH0",
  "has_more": true
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No, scroll/next
5. Yes
6. No
7. Client requests a large list
8. Paginate?
9. Server fetches all N rowstime O(N), memory O(N)slow, DoS risk
10. Cap page size k, enforce maxlimit
11. Need random page access?
12. Offset paginationmem O(k), time O(offset+k)
13. Cursor/keyset paginationmem O(k), time O(log N + k)
14. Return k items + page meta
15. Return k items + next_cursor
16. More pages?
17. Done: total work Theta(N),peak per-request O(k)

