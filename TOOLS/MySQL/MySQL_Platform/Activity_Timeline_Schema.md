---
id: "TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Activity Timeline Schema

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In MySQL/InnoDB the primary key *is* the table's physical order, so activity feeds, carts, preferences, leaderboards and funnels are designed by choosing keys and composite indexes that make the one hot query a short range scan — and by keeping analytics off the transactional path.

### 1. The big picture
"Timeline" and "activity" schemas — events per user, shopping carts, key-value preferences, soft-deleted accounts, undo history, trending items, funnels, leaderboards — recur in every product. They look different, but all are solved by the same toolkit: a clustered primary key that groups rows read together, secondary indexes that match the `WHERE` + `ORDER BY` shape, and a deliberate split between OLTP writes and analytical reads.

MySQL specifics matter: InnoDB stores rows in the primary-key B+tree (clustered index); every secondary index entry carries the primary key; MySQL has no partial indexes but 8.0 has functional indexes, generated columns and descending indexes; window functions and CTEs exist from 8.0.

### 2. Core concepts and vocabulary
- **Clustered index** — the PK B+tree that holds the row data; rows are ordered by PK.
- **Secondary index** — separate B+tree whose leaf stores indexed columns + the PK.
- **Covering index** — contains every column the query needs; no row lookup.
- **Composite index / leftmost prefix** — an index on (a, b, c) serves filters on a, a+b, a+b+c.
- **Generated column** — a column computed from others (`VIRTUAL` or `STORED`) and indexable.
- **Soft delete** — marking rows deleted (`deleted_at`) instead of removing them.
- **Append-only event table** — immutable rows, one per action.
- **Keyset (seek) pagination** — `WHERE (ts, id) < (?, ?) ORDER BY ts DESC, id DESC LIMIT n`.
- **Read replica** — asynchronous copy used for reads and analytics.
- **Rollup/summary table** — pre-aggregated counts maintained by jobs.

### 3. How it works, step by step — choosing the key
1. Write the dominant query in words: "latest 50 events for user X", "top 100 scores", "items by category in last 24 h".
2. Put the equality column(s) first and the ordering/range column last in the index: `(user_id, created_at, id)`.
3. Make the PK match the main access path when rows are always accessed per owner: `PRIMARY KEY (user_id, created_at, event_id)` clusters a user's events together; a monotonically increasing `AUTO_INCREMENT` PK keeps inserts append-only on a single hot page (fast, but spreads a user across the table).
4. Keep the PK short — it is copied into every secondary index.
5. Add secondary indexes only for known queries; each is another write.
6. Decide where analytics runs: read replica, summary table, or export to a warehouse.

### 4. Worked example
```sql
-- Activity timeline
CREATE TABLE user_events (
  user_id    BIGINT UNSIGNED NOT NULL,
  event_id   BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  event_type VARCHAR(32) NOT NULL,
  item_id    BIGINT UNSIGNED NULL,
  created_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
  payload    JSON NULL,
  PRIMARY KEY (user_id, created_at, event_id),
  KEY ix_event_id (event_id),                 -- AUTO_INCREMENT must be the first column of some index
  KEY ix_type_time (event_type, created_at)
) ENGINE=InnoDB;

-- Latest 50 for a user (single range scan, no sort)
SELECT event_id, event_type, item_id, created_at
FROM user_events
WHERE user_id = ? AND (created_at, event_id) < (?, ?)   -- keyset cursor
ORDER BY created_at DESC, event_id DESC
LIMIT 50;
```
```sql
-- Soft delete that keeps "unique among live users" enforceable (no partial indexes)
CREATE TABLE users (
  id         BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  email      VARCHAR(255) NOT NULL,
  deleted_at DATETIME NULL,
  active_email VARCHAR(255) AS (IF(deleted_at IS NULL, email, NULL)) STORED,
  UNIQUE KEY uq_active_email (active_email)     -- NULLs are allowed many times
);
```
```sql
-- Leaderboard: top 100 + my rank
SELECT user_id, score FROM scores ORDER BY score DESC, user_id LIMIT 100;      -- index (score, user_id)
SELECT COUNT(*) + 1 AS my_rank FROM scores
WHERE score > (SELECT score FROM scores WHERE user_id = ?);                    -- index range count
```
```sql
-- Funnel with per-step drop-off (8.0)
WITH steps AS (
  SELECT user_id,
         MAX(event_type = 'view')      AS v,
         MAX(event_type = 'add_cart')  AS a,
         MAX(event_type = 'checkout')  AS c,
         MAX(event_type = 'pay')       AS p
  FROM user_events
  WHERE created_at >= NOW() - INTERVAL 7 DAY
  GROUP BY user_id)
SELECT SUM(v) views, SUM(a) carts, SUM(c) checkouts, SUM(p) paid FROM steps;
```
Use `EXPLAIN ANALYZE` to confirm each statement shows an index range scan, `Using index` or an index lookup on the expected key, not `Using filesort` on millions of rows.

### 5. Under the hood
- **Clustered locality.** With `PRIMARY KEY (user_id, created_at, …)` a user's timeline is contiguous pages: one seek then sequential reads. The price: inserts land in many pages (page splits, more random I/O) rather than only at the tail. For firehose ingestion with rare per-user reads, `AUTO_INCREMENT` PK + secondary index `(user_id, created_at)` is the other choice (secondary lookups then touch the base table — one extra lookup per row unless covering).
- **Soft-delete uniqueness.** MySQL unique indexes treat NULL as distinct, so `UNIQUE(email, deleted_at)` fails to prevent two live rows (both `deleted_at IS NULL`). A generated column or sentinel (`deleted_at DEFAULT '9999-12-31'`) restores the constraint.
- **Preferences as key-value.** `PRIMARY KEY (user_id, pref_key)` makes "all prefs for a user" a single range scan; or store a `JSON` column on the user row if attributes are always read together (loses per-key indexing; use a generated column when you must query one key).
- **Cart merge** at login: do it in one transaction — `INSERT … ON DUPLICATE KEY UPDATE qty = qty + VALUES(qty)` from the guest cart into the user cart keyed `(cart_id, sku)`, then delete the guest cart; lock ordering by cart id prevents deadlocks, and an idempotency key prevents double-merge on retry.
- **Trending in a rolling window.** Counting raw events per query is expensive. Maintain 1-minute bucket counters (`PRIMARY KEY (item_id, bucket)`), query the last 1,440 buckets summed, or keep a sorted set in a cache and periodically write snapshots to MySQL.
- **Undo.** Store per-action inverse/previous values in an append-only `action_log` with a stack pointer per user; cheaper than event sourcing but only reconstructs what you logged. Event sourcing gives full replay at the cost of rebuild logic and storage.
- **Mixed OLTP/analytics.** Long scans on the primary evict hot pages from the buffer pool and hold old row versions (undo purge lag). Move them to a replica, a summary table, or a columnar engine (HeatWave, warehouse). Use `SET SESSION TRANSACTION READ ONLY` and resource groups where available.

### 6. Performance and design implications
- **Index selectivity and order**: equality columns first, then ranges; a range in the middle blocks use of later columns for seeking.
- **Avoid `OFFSET`** for deep timelines — use keyset pagination.
- **Narrow rows**: keep large `payload` blobs out of hot tables, or in a side table.
- **Retention**: partition by time (`PARTITION BY RANGE (TO_DAYS(created_at))`) so dropping old data is a metadata operation, and note that MySQL requires the partition column be part of every unique key.
- **Watch**: `sys.schema_unused_indexes`, `performance_schema.events_statements_summary_by_digest`, `SHOW ENGINE INNODB STATUS`, replication lag.
- **Write amplification**: each secondary index multiplies the write cost; batch inserts (hundreds of rows per statement) and avoid per-row autocommit.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN ANALYZE` | Actual plan and timings (8.0.18+) | Verify index use |
| `INSERT … ON DUPLICATE KEY UPDATE` | Upsert | Cart merge, counters |
| Generated columns + index | Index an expression | Soft delete uniqueness, JSON key |
| Invisible index (`ALTER … INVISIBLE`) | Test dropping an index safely | Index cleanup |
| `PARTITION BY RANGE` | Time-based retention | Event tables |
| `innodb_buffer_pool_size` | Cache size | Working-set fit |
| Read replicas / `SELECT … FOR SHARE` | Offload reads, lock semantics | Analytics, consistency needs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Composite PK `(user_id, ts, id)` | Insert-heavy, multi-GB/s firehose, rarely read per user |
| `AUTO_INCREMENT` PK + secondary index | Per-user range reads dominate |
| JSON column for preferences | You must filter/sort on individual keys at scale |
| Soft delete | Regulations demand physical deletion — add a purge job |
| Summary tables | Exact real-time counts are mandatory |

### 9. Common mistakes
- `UNIQUE(email, deleted_at)` for soft delete → duplicates of NULL → generated column or sentinel.
- `ORDER BY created_at DESC LIMIT 20 OFFSET 100000` → scans and discards → keyset pagination.
- Index per column "just in case" → slow writes, big buffer pool → index per real query.
- UUIDv4 as clustered PK → random inserts and fat secondary indexes → UUIDv7/ordered or `BIGINT`.
- Running reports on the primary → buffer pool churn and lock/undo contention → replica or summary tables.
- Cart merge without transaction/idempotency → lost or doubled items → single transaction, upsert, idempotency key.

### 10. 🎯 Interview kit
**30-second answer:** I design from the hot query: equality columns first then the ordering column, in the clustered PK when rows are read per owner, secondary indexes for other paths, keyset pagination, summary tables for rollups, and a replica for analytics. Soft-delete uniqueness uses a generated column because MySQL has no partial indexes.

**Structure a longer answer:** access patterns → key/index design → write path and constraints → scale/retention → analytics isolation.

**Likely follow-ups:**
- *Leaderboard in one round trip?* → index `(score DESC, user_id)`; top 100 via LIMIT; rank via range count.
- *Funnel on 10 M events?* → conditional aggregation per user, bounded time range, pre-aggregated daily table.
- *Undo vs event sourcing?* → inverse-log is smaller and simpler; event sourcing is replayable but heavier.
- *Why not UUID PK?* → random inserts, large index entries.

**Signals interviewers look for:** index-order reasoning, clustered-index awareness, trade-off statements, practical MySQL workarounds.

### 11. Check yourself
- [ ] Write the DDL and query for a per-user activity feed with keyset pagination.
- [ ] Implement soft delete with unique active emails in MySQL.
- [ ] Explain what a secondary index stores.
- [ ] Design a trending query that doesn't scan raw events.
- [ ] Choose between composite PK and auto-increment PK for a firehose.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Intermediate] [How would you design and index a schema for storing user preferences as key-value pairs while keeping 'get all prefs for a user' a single fast lookup?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33501_How_would_you_design_and_index_a_schema_for_storing_user_pre.md>)
- [Intermediate] [Explain how you'd design a schema to represent product variants (size/color/SKU) with inventory, and keep queries simple for both the catalog page and checkout.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33198_Explain_how_you_d_design_a_schema_to_represent_product_varia.md>)
- [Deep] [Explain how you'd model a tags-per-article schema so filtering by multiple tags (AND semantics) stays index-friendly instead of degenerating into many self-joins.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33471_Explain_how_you_d_model_a_tags-per-article_schema_so_filteri.md>)
- [Deep] [How would you design a schema and query to compute month-over-month growth per product, handling months where a product had zero sales?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33537_How_would_you_design_a_schema_and_query_to_compute_month-ove.md>)
- [Deep] [How would you design a schema and query to compute cohort retention (users active in week N after signup) efficiently on tens of millions of events?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33171_How_would_you_design_a_schema_and_query_to_compute_cohort_re.md>)
- [Deep] [You need to enforce that at most one row per user has status='active' — how do you do this at the schema level in MySQL 8 given no partial-index support?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33230_You_need_to_enforce_that_at_most_one_row_per_user_has_status.md>)
- [Deep] [How would you design a schema and access pattern for storing per-request API metrics (latency histograms) in MySQL cheaply, and when do you push it to a TSDB instead?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33275_How_would_you_design_a_schema_and_access_pattern_for_storing.md>)
- [Deep] [How would you design storage for versioned documents (like a wiki) so fetching the latest is O(1) and fetching a diff between two versions is efficient?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33309_How_would_you_design_storage_for_versioned_documents_like_a_.md>)

## Related concepts

- [Modeling Relationships and Schemas](<../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Experiment Metrics in SQL](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>) *(SQL Fundamentals)*
- [Idempotency Key Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>) *(SQL Fundamentals)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.1 Designing a schema using judgment
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — and C↑ or C↓ to represent clustering columns. Lines are shown entering tables or between tables to
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — comparision with the original design is shown in Figure 5-10.(#indexsplit001.htmlp185) While the

