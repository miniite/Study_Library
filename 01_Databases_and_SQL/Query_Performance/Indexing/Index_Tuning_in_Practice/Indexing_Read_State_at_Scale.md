---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Indexing_Read_State_at_Scale"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Indexing Read State at Scale

**L5 Expert** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** "Read state" features — unread counts, inboxes, leaderboards, message history, search-as-you-type — need each access pattern backed by an index (or a purpose-built structure) shaped for exactly that read, and the right choice shifts from B-tree indexes to counters and external stores as scale grows.

### 1. The big picture
Classic indexing asks "which index makes this query fast?". Product features at scale ask several questions of the *same* entity at once: "latest threads", "only unread", "search subject", "my rank", "unread badge". One B-tree can't serve all of them; piling indexes on one table inflates write cost, and some questions (exact rank, fuzzy match) are not what B-trees do well.

The skill is to enumerate the **access patterns** (with frequency, freshness and consistency needs), give each the cheapest adequate structure, and decide where the source of truth lives. This lesson works the common patterns: inbox/unread, notifications with bulk mark-as-read, chat history, count-by-status widgets, leaderboards, and fuzzy/search-as-you-type.

### 2. Core concepts and vocabulary
- **Access pattern** — a specific query shape: filters, ordering, limit, consistency, frequency.
- **Partial (filtered) index** — indexes only rows meeting a predicate, e.g. unread items.
- **Read cursor / watermark** — a per-user "last_read_id/time" instead of per-item read flags.
- **Counter table** — maintained aggregate (e.g. unread count per user) updated on writes.
- **Fan-out on write vs read** — computing per-recipient state when the event happens vs when someone reads.
- **Keyset pagination** — "give me items older than this (time,id) cursor".
- **Trigram index** — substring/similarity index over 3-character grams.
- **Inverted index** — term → posting list; basis of full-text search.
- **Phonetic code** — Soundex/Metaphone key mapping similar-sounding names to the same value.
- **Source of truth** — the store whose value wins; others are derived and may lag.

### 3. How it works, step by step
**Inbox: "most recent threads", "unread only", "search by subject" (millions of rows)**
1. *Recent threads:* `(user_id, last_message_at DESC)` on a **thread** table (one row per thread per user), keyset-paged. Not on message rows — avoid grouping at read time.
2. *Unread only:* a partial index `(user_id, last_message_at DESC) WHERE unread` — tiny, because unread is a small fraction, and writes touch it only on state change.
3. *Unread badge:* a `unread_count` per user maintained transactionally or via counter table; don't `COUNT(*)` on every page load.
4. *Subject search:* trigram or full-text index (`GIN (to_tsvector(...))`), or a search service; keep it off the hot OLTP index set.

**Per-user unread state (notifications), bulk mark-as-read**
- *Per-item flag + partial index* `(user_id, id) WHERE read_at IS NULL`: simple; bulk "mark all read" updates many rows (write amplification, bloat).
- *Last-read cursor*: `user_state(user_id, last_read_id)`; unread = `id > last_read_id` → O(1) mark-all-read, but cannot represent "read #5 but not #4" without extras.
- *Counters*: `unread_count` in `user_state`, incremented on insert, reset on mark-all-read; exact counts cheap.
Combine cursor + counter + optional exception set for individual toggles.

**Chat history:** index `(conversation_id, created_at DESC, id DESC)`.
- Last 50: seek to the end of the conversation's range, read 50.
- Scroll up: keyset `WHERE conversation_id = ? AND (created_at, id) < (?, ?) ORDER BY created_at DESC, id DESC LIMIT 50`.
- Jump to a search hit: fetch around its `(created_at,id)` with two keyset queries (older/newer); search hits come from a text index.

**Leaderboard:** top-N from `(score DESC)`; exact rank = `1 + COUNT(*) WHERE score > my_score` (fine to ~millions with an index, O(n) otherwise); at 100 M users use a sorted-set store (O(log n) rank) with the database as the durable record, plus separate structures for friends' ranks.

### 4. Worked example
```sql
-- PostgreSQL: notifications with unread partial index and cursor
CREATE TABLE notification (
  id         bigint GENERATED ALWAYS AS IDENTITY,
  user_id    bigint NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  read_at    timestamptz,
  payload    jsonb,
  PRIMARY KEY (user_id, id)
);
CREATE INDEX ix_notif_unread ON notification (user_id, id DESC) WHERE read_at IS NULL;

-- 1) unread list (partial index; touches only unread rows)
SELECT id, payload FROM notification
WHERE user_id = :u AND read_at IS NULL
ORDER BY id DESC LIMIT 50;

-- 2) bulk mark as read, in batches to bound locks/WAL
UPDATE notification SET read_at = now()
WHERE user_id = :u AND read_at IS NULL
  AND id IN (SELECT id FROM notification WHERE user_id = :u AND read_at IS NULL
             ORDER BY id LIMIT 1000);

-- 3) count by status widget: exact via counter table (trigger/app maintained)
UPDATE status_counts SET n = n + 1 WHERE status = 'open';
```
Check with `EXPLAIN`: query (1) should be an Index Scan on `ix_notif_unread` with no sort. Note the partial index shrinks as items are read, so it stays small; writes to `read_at` remove entries (deletes from the partial index). If you update `read_at` for millions of rows per day, vacuum pressure rises — the cursor design avoids this.

### 5. Under the hood
- **Why partial indexes shine for state.** The "unread" set is a small, churning subset. A partial index holds only that subset: smaller cache footprint, shallower tree, and no writes for rows that never become unread. Cost: the query predicate must imply the index predicate (the planner matches `read_at IS NULL` syntactically).
- **Counts are expensive in MVCC row stores.** `COUNT(*)` must verify visibility of every qualifying row (index-only scans help only with a fresh visibility map). Options for "count by status" widgets:
  | Option | Accuracy | Cost |
  |---|---|---|
  | Triggered/app-maintained counter table | exact | extra write per state change; row-level contention on hot counters (shard counters) |
  | Partial index per status + `COUNT(*)` | exact | still O(rows of that status) per read |
  | Approximate (`reltuples`, HLL, sampling, cached count) | ±% | near-free; fits dashboards |
  For a real-time UI that tolerates small drift, approximate or cached beats exact; for billing badges use a counter.
- **Race conditions in denormalized state.** Counters and cursors need atomic updates (`UPDATE … SET n = n + 1`), idempotent consumers when driven by events, and periodic reconciliation jobs that recompute from the source of truth.
- **Fuzzy matching options.**
  | Approach | Latency | Index size | Notes |
  |---|---|---|---|
  | Trigram (`pg_trgm` GIN/GiST) | ms–tens of ms | large (several grams per string) | substring + similarity; false positives need re-rank |
  | Phonetic codes (precomputed, B-tree) | sub-ms | small | only sound-alike; language-specific; needs precompute/maintenance |
  | Search service (Elasticsearch/OpenSearch) | ms | large, separate cluster | ranking, typo tolerance, facets; eventual consistency, ops cost |
- **Search-as-you-type** needs prefix matching: a B-tree with `text_pattern_ops` on a normalized column (low latency, exact prefix, no typos), n-gram/edge-n-gram index for typo tolerance, or search service. Freshness: B-tree/trigram are transactional; search tiers lag by seconds.
- **Where to draw the DB vs search boundary.** Keep *transactional filters, keyed lookups, and simple prefix* in the database; move *ranking, typo tolerance, synonyms, facets, huge corpora* to a search tier fed by CDC/outbox; define reconciliation and ownership. Over a three-year horizon account for team skills and operating cost, not only features.

### 6. Performance and design implications
- **Write path cost:** every extra index/structure is paid on each message/notification insert. For 50 M users, prefer cursors/counters (O(1) writes) over per-item flags for bulk operations.
- **Hot users:** celebrity feeds or whale tenants break uniform assumptions; keyset pagination plus per-user partitioning or pre-materialised feeds keeps latency bounded.
- **Isolation per user:** shard/partition by `user_id` so each user's reads touch one shard and one contiguous index range.
- **Graceful degradation:** serve cached/approximate counts when the counter store is slow; mark-as-read can be asynchronous (accept brief re-display).
- **Observability:** track p95 of each read path separately; alert on plan changes for the hot queries.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … WHERE read_at IS NULL` | Partial index | Small hot subset |
| `CREATE EXTENSION pg_trgm; CREATE INDEX … USING gin (col gin_trgm_ops)` | Trigram index | Substring / similarity |
| `CREATE INDEX … USING gin (to_tsvector('simple', col))` | Full-text index | Word search |
| `text_pattern_ops` | Prefix `LIKE` on B-tree | Autocomplete |
| Row-value comparison `(a,b) < (?,?)` | Keyset predicate | Stable scroll |
| Counter table + `UPDATE … SET n = n + 1` | Maintained aggregate | Badges and widgets |
| Redis `ZADD/ZRANK/ZREVRANGE` | Sorted-set ranking | Leaderboards at scale |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Per-item read flag + partial index | Users bulk-mark thousands of items often |
| Last-read cursor + counter | Need individual unread toggles (add exceptions) |
| Trigram/FTS in Postgres | Need ranking, typo tolerance, multilingual analysis at large scale |
| External search tier | Strong read-after-write consistency required without extra work |
| Counter tables | Hot counters cause row contention (shard the counter) |
| Exact rank via count | 100 M+ users and frequent rank reads |

### 9. Common mistakes
- **`COUNT(*)` per request for badges** → slow and lock-free but expensive → maintain counters or approximate.
- **Flag per row + giant `UPDATE … SET read=true`** → bloat, long locks → batch, or cursor design.
- **Indexing `read_status` alone** → half the table matches → partial or composite with `user_id`.
- **OFFSET for infinite scroll** → duplicates/skips and slow deep pages → keyset.
- **Using trigram for all search** → huge indexes and noisy ranking → define quality needs; consider a search tier.
- **No reconciliation of derived counters** → drift → periodic recompute and idempotent updates.
- **Making the cache the source of truth for rank** → loss on failure → persist scores in the database.

### 10. 🎯 Interview kit
**30-second answer:** I list the access patterns and give each its own cheap structure: composite indexes for ordered lists with keyset pagination, partial indexes for small hot subsets like unread, counters or cursors for badges and bulk mark-as-read, trigram or a search service for fuzzy text, and a sorted-set store for ranks. The database stays the source of truth.

**Structure a longer answer:** patterns and SLAs → index per pattern with write cost → derived state (counters/cursors) and consistency → search boundary → hot-key/skew handling → degradation and reconciliation.

**Likely follow-ups:**
- *Unread per-user: table, cursor, or counts?* → Cursor for bulk, counter for badge, partial index for listing; reconcile.
- *Count-by-status for real-time UI?* → Counter table if exact needed; approximate/cached if tolerance.
- *Search-as-you-type?* → Prefix B-tree (cheap, exact), trigram (substring/fuzzy), search service (best UX, ops cost).
- *Leaderboard with top-N, my rank, friends' rank?* → No single index; sorted-set for global rank, friend list intersection or per-friend lookups, DB as truth.

**Signals interviewers look for:** pattern-by-pattern thinking, explicit write-cost accounting, source-of-truth awareness, graceful degradation.

### 11. Check yourself
- [ ] Design indexes for inbox recency, unread-only and subject search, and state each one's write cost.
- [ ] Compare cursor vs per-row flag for "mark all as read".
- [ ] Explain why exact rank is hard on a B-tree and give two fixes.
- [ ] Choose between trigram, phonetic codes and a search service for "did you mean" on usernames.
- [ ] Describe how to reconcile a drifting counter table.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you design indexes for an inbox listing that needs 'most recent thread', 'unread only', and 'search by subject' at a few million rows?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20130_How_would_you_design_indexes_for_an_inbox_listing_that_needs.md>)
- [Deep] [You need fuzzy matching for a 'did you mean' feature on names. Compare a trigram index, precomputed phonetic codes, and delegating to a search service on cost and latency.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20154_You_need_fuzzy_matching_for_a_did_you_mean_feature_on_names_.md>)
- [Deep] [You need to support 'search as you type' across a products table. Compare a trigram index, a prefix B-tree, and delegating to a search service in terms of latency, freshness, and ops cost.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19684_You_need_to_support_search_as_you_type_across_a_products_tab.md>)
- [Deep] [A frequently-run 'count by status' widget is slow. Compare maintaining a triggered counter table, a partial index per status, and an approximate count. Which fits a real-time UI?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19700_A_frequently-run_count_by_status_widget_is_slow_Compare_main.md>)
- [Deep] [How would you design indexing and query patterns for a 'seen/unseen notifications' feature where each user reads their own unread set constantly and marks-as-read in bulk?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19752_How_would_you_design_indexing_and_query_patterns_for_a_seen_.md>)
- [Deep] [You must support fuzzy matching for a 'did you mean' feature over usernames. Compare a trigram index, precomputed phonetic codes, and an external search service on latency and index size.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19754_You_must_support_fuzzy_matching_for_a_did_you_mean_feature_o.md>)
- [Deep] [Design the read path for a chat app's message history: load last 50 by conversation, scroll up for older, jump to a search hit. Specify the indexes and how each access pattern uses them.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19760_Design_the_read_path_for_a_chat_app_s_message_history_load_l.md>)
- [Deep] [You need 'threads with unread messages for a user' where unread state is per-user. Would you index a per-user read-state table, denormalize a last_read cursor, or maintain unread counts? Discuss.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19837_You_need_threads_with_unread_messages_for_a_user_where_unrea.md>)

## Related concepts

- [Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>) *(High Level Design)*
- [Exact vs Approximate Counting](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Exact_vs_Approximate_Counting.md>) *(Complexity Analysis)*

## References

- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › What Will We Want?
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Tuning Scores
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › The No-Maintenance Columnstore Index
- *Data Engineering Design Patterns* — Bartosz Konieczny — T › U
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Q

