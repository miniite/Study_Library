---
type: "practice"
difficulty: "Intermediate"
---

# How would you design the schema for a URL shortener like Bitly?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Design the database schema for a URL shortener (Bitly-style): take a long URL, produce a short code, and on visiting short.ly/abc123 redirect to the original. Interviewers often use this to slide from schema into light system-design, so I should nail the tables and speak to the interesting decisions: how the short code is generated, uniqueness, the redirect read-path, analytics, expiry, and scale.

What the interviewer is probing for (intermediate):

The core url_mappings table and the choice of the short code as the natural lookup key.
Short-code generation strategy — auto-increment ID + Base62 encode vs. random string vs. hash — and the trade-offs (collisions, guessability, length).
The read-heavy nature (redirects vastly outnumber creations) and how the schema + caching serve it.
Analytics (click tracking) and lifecycle (expiry, custom aliases, per-user ownership).

Clarifying questions I'd raise: Do we need custom aliases (vanity codes)? Per-user accounts and dashboards? Click analytics (counts only, or per-click detail with geo/referrer)? Expiring links? Expected scale (writes/day, redirects/sec)? I'll design for: accounts + custom alias + analytics + optional expiry, and call out scale choices.

Plan: entities → short-code generation deep-dive → core schema → read path/caching → analytics table → expiry/lifecycle → scale/sharding notes → summary.

Core entities
users — owner of links (optional for anonymous shortening, required for dashboards).
url_mappings — the heart: short_code → long_url, plus owner, timestamps, expiry, flags.
click_events (optional, for detailed analytics) — one row per redirect, with referrer/geo/UA/time.

The whole product hinges on one lookup: given short_code, find long_url. So short_code must be UNIQUE and indexed — it's effectively the natural key of the read path. I keep a separate numeric surrogate PK id for internal FKs and for the ID-based code generation described next.

The key decision: generating the short code

Three strategies; I'd pick #1 for scale and explain why:

1. Auto-increment ID → Base62 encode (recommended). Insert the row, take the sequential id (a BIGINT), and encode it into Base62 ([0-9a-zA-Z], 62 symbols). Base62 gives short codes: 62^7 ≈ 3.5 trillion combinations in 7 characters. Because IDs are unique by construction, the encoded code is collision-free by design — no retry loop needed. Downsides: codes are somewhat guessable/sequential (mitigate by seeding the counter high, using a scrambled/offset encoding, or Sqids/Hashids to obfuscate). This is the classic, scalable answer.

2. Random string + uniqueness check. Generate a random 7-char Base62 string, INSERT ... ON CONFLICT/catch the unique-violation and retry. Non-guessable, but needs collision handling; at low fill ratios collisions are rare, but they grow as the keyspace fills. Simpler mentally, extra round-trips under contention.

3. Hash the long URL (e.g. MD5/SHA, take first N chars). Deterministic (same URL → same code, natural dedup), but truncation causes collisions, and it can't produce two different codes for the same URL if a user wants that. Rarely the best default.

Whichever I choose, short_code gets a UNIQUE constraint so the DB is the final arbiter of uniqueness even under race conditions. Custom aliases slot into the same column: a user-supplied short_code just competes for the same unique index (reject on conflict).

Read path and caching

Redirects are the dominant, latency-sensitive operation (often 100:1 or more vs. creates). The schema supports this with the unique index on short_code (a single indexed point lookup). In front of the DB you put a cache (Redis/Memcached) keyed by short_code → long_url; the DB is the source of truth, the cache absorbs the read storm. Because rows are essentially immutable after creation (the mapping doesn't change), caching is easy and cache invalidation is trivial (only on delete/expiry). This is worth stating: the immutability of the mapping is what makes aggressive caching safe.

Analytics

Two levels:

Cheap counter: a click_count column on url_mappings, incremented per redirect (or, better, buffered/aggregated to avoid a hot-row write on every hit — increment in Redis and flush periodically).
Detailed events: a click_events table, one row per redirect with clicked_at, referrer, country, user_agent, ip_hash. This is append-only and high-volume, so it's a candidate for time-partitioning, a separate analytics store, or async ingestion — never let analytics writes block the redirect. Dashboards then aggregate (GROUP BY date, top referrers, etc.).
Lifecycle: expiry & custom aliases
Expiry: expires_at TIMESTAMPTZ NULL. On lookup, if now() > expires_at, treat as 404/410. A background job (or TTL in a cache/expires_at index) purges expired rows.
Soft delete / disable: is_active BOOLEAN so a link can be disabled without losing analytics history.
Custom alias: stored in the same short_code column; unique index enforces "not taken".
Scale notes (shows senior thinking)

At Bitly scale (billions of links), a single relational table strains. Talking points: (a) the short_code unique index is the critical structure; (b) shard by short_code hash or by ID range; (c) use the Base62 approach so ID generation itself scales (distributed ID via ranges/Snowflake to avoid a single sequence bottleneck); (d) the store can even be a KV/NoSQL (short_code as partition key) because the access pattern is a pure point lookup — a strong candidate to mention. click_events scales separately as an append-only/analytics pipeline.

#### Common mistakes
/ follow-ups
Forgetting the UNIQUE on short_code → duplicate codes, ambiguous redirects.
Doing a synchronous per-click UPDATE ... SET click_count = click_count+1 on the hot mapping row → write contention; buffer it.
Ignoring guessability of sequential codes → obfuscate.
Not handling custom-alias collisions with the same unique index.

#### Follow-ups
"How do you guarantee code uniqueness across shards?" (distributed ID ranges). "How do you handle the same long URL shortened twice?" (allow it — different codes — or dedup by hashing, your call). "Rate limiting / abuse (phishing URLs)?" (validation + blocklist table).
Summary — what a strong answer emphasizes

One central url_mappings table with a surrogate id PK and a UNIQUE-indexed short_code as the natural lookup key; generate the code via auto-increment ID → Base62 for collision-free, short, scalable codes (mention random+retry and hashing as alternatives with their trade-offs). Add owner (user_id), timestamps, expires_at, is_active, and support custom aliases through the same unique column. Because the mapping is immutable and read-heavy, front it with a cache; keep click analytics either as a buffered counter or a separate append-only click_events table so redirects stay fast. Close with sharding / distributed-ID / KV-store options for Bitly-scale.

**Comparison:**

```
Code-generation strategy	Collision handling	Guessable?	Length	Verdict
Auto-increment id → Base62	none needed (unique by construction)	sequential (obfuscate)	short (~7 chars for trillions)	Recommended, scales
Random Base62 + uniqueness check	retry on unique-violation	no	short	Simple; extra round-trips as keyspace fills
Hash(long_url) truncated	truncation collisions possible	no	short	Natural dedup but collision-prone
Concern	Schema element
Fast redirect lookup	UNIQUE index on short_code + cache
Custom / vanity alias	same short_code column, unique constraint
Expiry	expires_at + partial index + purge job
Enable/disable without data loss	is_active flag
Analytics	click_count (buffered) + click_events (append-only)
Scale	shard by short_code hash; distributed ID; KV store option
```

**Code:**

```

CREATE TABLE users ( user_id BIGSERIAL PRIMARY KEY, email TEXT NOT NULL UNIQUE, created_at TIMESTAMPTZ NOT NULL DEFAULT now() );

CREATE TABLE url_mappings ( id BIGSERIAL PRIMARY KEY, -- internal id (also feeds Base62 code) short_code VARCHAR(16) NOT NULL, -- the lookup key (e.g. 'aZ9kQ2b' or custom alias) long_url TEXT NOT NULL, user_id BIGINT REFERENCES users(user_id) ON DELETE SET NULL, -- NULL = anonymous is_custom BOOLEAN NOT NULL DEFAULT FALSE, is_active BOOLEAN NOT NULL DEFAULT TRUE, click_count BIGINT NOT NULL DEFAULT 0, -- buffered/aggregated counter expires_at TIMESTAMPTZ, -- NULL = never expires created_at TIMESTAMPTZ NOT NULL DEFAULT now(), CONSTRAINT uq_short_code UNIQUE (short_code) -- guarantees uniqueness incl. custom aliases );

-- Primary read path: point lookup by short_code (unique index above already serves it). CREATE INDEX idx_urlmap_user ON url_mappings(user_id); CREATE INDEX idx_urlmap_expires ON url_mappings(expires_at) WHERE expires_at IS NOT NULL;

-- Detailed analytics (append-only, high volume; partition by month at scale) CREATE TABLE click_events ( event_id BIGSERIAL PRIMARY KEY, mapping_id BIGINT NOT NULL REFERENCES url_mappings(id) ON DELETE CASCADE, clicked_at TIMESTAMPTZ NOT NULL DEFAULT now(), referrer TEXT, country CHAR(2), user_agent TEXT, ip_hash TEXT -- hashed for privacy, not raw IP ); CREATE INDEX idx_click_mapping_time ON click_events(mapping_id, clicked_at);

CODE
Copy

```python
# Short-code generation: auto-increment id -> Base62 (collision-free by construction)
ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"  # 62 chars

def encode_base62(n: int) -> str:
    if n == 0:
        return ALPHABET[0]
    s = []
    while n > 0:
        n, r = divmod(n, 62)
        s.append(ALPHABET[r])
    return "".join(reversed(s))

# Create flow (pseudocode):
#   row = INSERT INTO url_mappings(short_code='', long_url=?) RETURNING id
#   code = encode_base62(row.id + OFFSET)      # OFFSET to avoid tiny/guessable codes
#   UPDATE url_mappings SET short_code = code WHERE id = row.id
# Redirect flow:
#   long = cache.get(code) or  SELECT long_url FROM url_mappings
#          WHERE short_code=? AND is_active AND (expires_at IS NULL OR expires_at>now())
#   302 -> long ; async increment click_count / insert click_event
```

**Example input / output:**

```text
CREATE:
INPUT : POST /shorten { long_url: "https://example.com/very/long/path" }
STEPS : INSERT -> id=1000 ; encode_base62(1000)="g8" (with OFFSET->longer in prod)
        UPDATE short_code='g8'
OUTPUT: 201 { short_url: "https://short.ly/g8" }

REDIRECT:
INPUT : GET https://short.ly/g8
STEPS : cache miss -> SELECT long_url WHERE short_code='g8' AND is_active
        AND (expires_at IS NULL OR expires_at>now())  -> hit
        cache.set('g8', long_url); async: click_count++ / INSERT click_event
OUTPUT: HTTP 302 Location: https://example.com/very/long/path

EXPIRED / DISABLED:
INPUT : GET https://short.ly/oldcode  (expires_at < now)
OUTPUT: HTTP 410 Gone (or 404)

CUSTOM ALIAS COLLISION:
INPUT : POST /shorten { long_url:..., alias:"launch2026" } when already taken
OUTPUT: 409 Conflict (unique constraint uq_short_code)
```

**Data:**

```text
users
 user_id | email
 --------+------------------
 1       | asha@example.com

url_mappings
 id   | short_code | long_url                              | user_id | click_count | expires_at
 -----+------------+---------------------------------------+---------+-------------+-----------
 1000 | g8         | https://example.com/very/long/path... | 1       | 42          | NULL
 1001 | launch2026 | https://example.com/campaign          | 1       | 1500        | 2026-12-31   <- custom alias, is_custom=true

click_events  (one row per redirect)
 event_id | mapping_id | clicked_at           | referrer        | country
 ---------+------------+----------------------+-----------------+--------
 5        | 1001       | 2026-07-07T10:00:00Z | twitter.com     | IN
 6        | 1001       | 2026-07-07T10:01:12Z | (direct)        | US
```

**Flow:**

1. yes
2. no
3. found
4. not found / expired
5. GET short.ly/:code
6. in cache?
7. 302 -> long_url
8. SELECT long_url FROMurl_mappingsWHERE short_code=:codeAND is_activeAND expires_at IS NULL OR >now
9. cache.set code -> long_url
10. 404 / 410
11. async: click_count++ / INSERTclick_event
12. owns
13. tracked by
14. USERS
15. bigint
16. user_id
17. PK
18. text
19. email
20. URL_MAPPINGS
21. bigint
22. id
23. PK
24. varchar
25. short_code
26. UK
27. text
28. long_url
29. bigint
30. user_id
31. FK
32. timestamptz
33. expires_at
34. bigint
35. click_count
36. CLICK_EVENTS
37. bigint
38. event_id
39. PK
40. bigint
41. mapping_id
42. FK
43. timestamptz
44. clicked_at
45. text
46. referrer
47. char
48. country

