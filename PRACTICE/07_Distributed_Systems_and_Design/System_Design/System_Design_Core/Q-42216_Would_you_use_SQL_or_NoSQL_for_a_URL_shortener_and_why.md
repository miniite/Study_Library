---
type: "practice"
difficulty: "Intermediate"
---

# Would you use SQL or NoSQL for a URL shortener and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pagination Strategies](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Pagination_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The interviewer wants to see whether you can pick a datastore from the access pattern and workload, not from a hype preference. "URL shortener" (like TinyURL/bit.ly) is deliberately a small, well-understood system, so the real probe is your reasoning: what are the reads/writes, the consistency needs, the scale, and how those map to SQL vs NoSQL trade-offs. A weak answer says "NoSQL because it scales." A strong answer says "here is the workload, here is what each store gives me, and here is the pragmatic choice — and I can justify either with the right justification."

Clarifying questions I'd ask first:

Scale: how many short URLs total and QPS? (10K links vs 100B links is a different problem.)
Read:write ratio? (Shorteners are extremely read-heavy, often 100:1 or more.)
Do we need analytics (click counts, geo, referrer) or just redirect?
Custom aliases and link expiry? Multi-region / global low latency?

#### Assumptions
for this answer: large scale (billions of URLs, tens of thousands of redirect QPS), read-heavy, redirect latency must be a few milliseconds, plus optional click analytics.

The core data model and access pattern

The primary object is tiny and its access is by exact key:

Write: create(short_code) -> long_url (plus created_at, owner, expiry).
Read: GET /{short_code} → look up the row by short_code → 301/302 redirect. This is a point lookup by primary key, the single most important operation, and it happens on the hot path of every redirect.

There are almost no joins, no multi-row transactions, no complex queries in the core flow. That single fact drives the whole decision: the workload is a giant key-value lookup with a modest write path.

What SQL gives you here

A relational store (Postgres/MySQL) is a completely legitimate default:

Strong consistency + unique constraints. A UNIQUE index on short_code prevents two long URLs from ever getting the same code — the collision problem is solved by the database, not by hand-rolled locking.
Auto-increment ID → code. If you generate codes by base-62 encoding an auto-increment primary key, you get short, collision-free, monotonically unique codes essentially for free. This is one of the cleanest known designs for a shortener.
Familiar, transactional, easy analytics. SELECT count(*) for clicks, joins to a users table, ad-hoc reporting — all trivial in SQL.
It scales further than juniors think: a single Postgres box with the row cached in RAM handles very high point-lookup QPS, and read replicas absorb the read-heavy traffic.

The limitation: a single SQL primary struggles once writes or dataset size exceed one machine. Then you must shard manually (by hash of short_code), and cross-shard uniqueness/ID generation gets fiddly. SQL doesn't horizontally scale writes as effortlessly as a distributed KV store.

What NoSQL gives you here

A distributed key-value / wide-column store (DynamoDB, Cassandra, or even Redis as the read layer) fits the access pattern like a glove:

The lookup is partition_key = short_code, which is exactly what these engines are optimized for — O(1)-ish, single-digit-ms, at any scale.
Horizontal scalability is built in. Partitioning/replication is automatic; billions of keys and huge write throughput spread across nodes with no manual sharding.
High availability / multi-region. Cassandra/DynamoDB give tunable consistency and active-active geo-replication, which matches "global redirects must be fast everywhere."

The trade-offs: no auto-increment, so you generate codes yourself (random base-62 + collision check, or a distributed ID service like a KGS/ticket-server or Snowflake). Ad-hoc queries and analytics are weaker; you model access patterns up front, and aggregations (click counts) usually go to a separate analytics pipeline or an atomic counter.

My recommended answer

For a large-scale shortener I lean NoSQL for the core mapping because the workload is a pure, massive, read-heavy key-value lookup that must stay fast globally, and horizontal scale + geo-replication come for free. Concretely:

Mapping store: DynamoDB/Cassandra keyed by short_code.
Read cache: Redis/CDN in front — most reads are a handful of hot links, so the cache absorbs the bulk of redirect QPS at sub-millisecond latency; the DB is the source of truth on miss.
ID/code generation: a Key Generation Service that pre-generates unique base-62 codes (or Snowflake-style IDs), avoiding both collisions and hot single-counter contention.
Analytics: clicks streamed to a separate system (Kafka → warehouse), not the hot path.

But I would explicitly say either is defensible. If scale is modest (a startup, millions not billions), I'd actually pick Postgres — the auto-increment→base-62 trick gives free unique codes, ACID guarantees, easy analytics, and one less distributed system to operate. Choose the boring, reliable tool until scale forces you off it. The senior signal is matching the tool to the actual numbers, and knowing that the real bottleneck (read latency) is solved by caching regardless of which persistent store you pick.

Summary — what a strong answer emphasizes
Identify the workload first: point lookup by key, extremely read-heavy, no joins/transactions.
NoSQL (KV/wide-column) wins on horizontal scale, global low latency, and matches the access pattern; cost is manual code generation and weaker ad-hoc analytics.
SQL wins on unique constraints, the auto-increment→base-62 code trick, ACID, and easy analytics; cost is manual sharding beyond one machine.
Real system uses a cache in front regardless — that, not the DB choice, is what makes redirects fast.
Correct final stance: NoSQL at hyperscale, SQL at modest scale — justify by numbers, not by fashion.

#### Common mistakes
/ follow-up probes: saying "NoSQL because scale" with no access-pattern reasoning; forgetting the collision problem in code generation; ignoring the read cache; not mentioning how you keep short_code unique across shards. A common follow-up: "How do you generate unique codes?" (KGS/counter/random+check) and "301 vs 302?" (302/307 if you want to keep analytics and allow changes; 301 is cached by browsers and skips your server).

**Comparison:**

```
Dimension	SQL (Postgres/MySQL)	NoSQL (DynamoDB/Cassandra)
Core access pattern (point lookup by code)	Great with PK/index	Great — this is its sweet spot
Unique code generation	Free via auto-increment → base62	Manual (KGS / random+collision check)
Horizontal write scale	Manual sharding needed	Built-in partitioning
Global low-latency / multi-region	Read replicas; geo is extra work	Native active-active replication
Consistency / constraints	Strong, ACID, UNIQUE constraint	Tunable (often eventual by default)
Analytics / ad-hoc queries	Easy (SQL, joins, aggregates)	Weaker; use separate pipeline
Ops complexity at scale	Higher (sharding)	Lower (auto-scale)
Best when	Modest scale, need ACID/analytics	Billions of links, global read-heavy
```

**Code:**

```

-- SQL design: auto-increment PK gives free unique codes via base-62 encoding CREATE TABLE urls ( id BIGSERIAL PRIMARY KEY, -- monotonically unique short_code VARCHAR(10) UNIQUE NOT NULL, -- base62(id), enforced unique long_url TEXT NOT NULL, owner_id BIGINT, created_at TIMESTAMPTZ DEFAULT now(), expires_at TIMESTAMPTZ ); CREATE INDEX idx_urls_code ON urls (short_code); -- point lookup on redirect

CODE
Copy

```python
# Free, collision-proof code from an auto-increment id (works for SQL path)
ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"  # base62
def encode(n: int) -> str:
    if n == 0:
        return ALPHABET[0]
    s = []
    while n:
        n, r = divmod(n, 62)
        s.append(ALPHABET[r])
    return "".join(reversed(s))
# encode(125) -> "21"; encode(999999) -> "4c91"
```

**Example input / output:**

```text
WRITE (shorten):
  POST /shorten { "long_url": "https://example.com/a/very/long/path" }
  -> id=999999 assigned -> encode(999999)="4c91"
  -> store {short_code:"4c91", long_url:"..."}
  <- 201 { "short_url": "https://sho.rt/4c91" }

READ (redirect, hot path):
  GET /4c91
  -> cache lookup "4c91"  (Redis) : HIT -> long_url in <1ms
     (on MISS) -> KV lookup partition_key="4c91" -> long_url, warm cache
  <- 302 Location: https://example.com/a/very/long/path
```

**Data:**

```json
// DynamoDB / Cassandra item — partition key is the short_code (point lookup)
{
  "short_code": "4c91",          // partition key
  "long_url":   "https://example.com/a/very/long/path?ref=abc",
  "owner_id":   "u_1042",
  "created_at": "2026-07-07T10:00:00Z",
  "expires_at": null,
  "clicks":     18342             // atomic counter, or kept in analytics store
}
```

**Flow:**

1. hit
2. miss
3. Write path
4. POST /shorten
5. Key Gen Servicebase62 code
6. (Persist mapping)
7. Client GET /4c91
8. Redis / CDN cache?
9. 302 redirect to long_url
10. (KV store: PK=short_code)
11. Warm cache

