---
type: "practice"
difficulty: "Intermediate"
---

# What is denormalization and when would you deliberately denormalize a schema?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer wants two things: a precise definition of denormalization, and — more importantly for an intermediate SDE — the judgment of when you'd deliberately do it. The companies tagged (Amazon, Netflix, Uber, Swiggy, CRED, PhonePe, Flipkart, Zomato) are all high-throughput, read-heavy systems, so the probe is really: "Do you understand the read-vs-write, storage-vs-latency trade-off, and can you name concrete situations where breaking 3NF is the right engineering call — and how you'd keep the duplicated data consistent?"

Clarifying framing: Denormalization is a deliberate introduction of redundancy into a schema that you could have kept fully normalized. It is not "being sloppy about normalization." The key tension: normalization optimizes writes and correctness (one source of truth, no update anomalies); denormalization optimizes reads (fewer joins, fewer round trips) at the cost of redundancy and the burden of keeping copies in sync.

Plan: (1) define it and contrast with normalization; (2) enumerate the concrete triggers — when to reach for it; (3) show the common techniques; (4) cover the cost side and how to keep data consistent; (5) give a crisp decision rule and follow-up probes.

What denormalization is

In a normalized (3NF/BCNF) schema, each fact lives in exactly one place, and you reconstruct wider views by joining tables. Denormalization intentionally stores some data redundantly — either duplicating a column into another table, pre-joining tables, or pre-computing aggregates — so that a common read can be served with fewer joins, fewer queries, or a single row lookup. You trade write complexity and storage for read latency and simplicity.

Concrete example: In an e-commerce OrderLines table you normally store only product_id. But the "order history" page shows the product name and price at time of purchase. Joining to Products on every render is extra work — and worse, the product's current price/name may have changed, so the join would show the wrong historical value. So you denormalize: store product_name and unit_price_at_purchase directly on the order line. This is both a performance win and a correctness requirement (you're snapshotting history).

When you would deliberately denormalize

Read-heavy, join-heavy hot paths. A feed, a product page, or a dashboard read millions of times per minute where the same 4-table join dominates latency. Pre-joining or duplicating a few columns removes the join from the critical path. This is why Amazon/Flipkart product pages and Netflix "continue watching" rows are denormalized.

Expensive aggregates read far more than written. A "like count", "follower count", or "order total" is read on every page load but changes occasionally. Storing a maintained counter (post.like_count) instead of COUNT(*)-ing a huge likes table on every read is a classic denormalization.

Historical / point-in-time snapshots. Invoices, order lines, and audit records must reflect values as they were, so you copy price, address, tax_rate at write time. Here denormalization is not optional — a live join would show today's values, which is wrong.

Cross-service / distributed data (NoSQL, microservices). In DynamoDB/Cassandra you model for the query: you can't join across partitions cheaply, so you duplicate data into each access pattern's table (single-table design). Uber/CRED-scale systems do this heavily.

Analytical / OLAP schemas. Star schemas (fact + dimension tables) are deliberately denormalized versus a fully normalized snowflake, because BI queries scan and aggregate, and wide flat rows are faster to scan than deep join chains.

Reducing round trips / N+1 problems. When an API would otherwise fire one query per row, embedding the needed field avoids the N+1 explosion.

Common techniques
Duplicating columns (copy customer_name onto orders).
Pre-joining / wide tables (a materialized order_summary combining order + customer + shipping).
Precomputed aggregates / counters (like_count, order_total).
Materialized views (DB-maintained pre-joined/aggregated results, refreshed on a schedule or incrementally).
Arrays / embedded documents (Postgres JSONB, MongoDB sub-documents) to co-locate related data.
The cost — and how to control it

The price of every duplicated fact is: more storage, slower/heavier writes (you now update N copies), and above all the risk of the copies drifting out of sync. You manage this with:

Application-level dual writes — but wrap them in a transaction so they stay atomic.
Database triggers — auto-update the counter/copy when the source changes (simple, but can hide write cost).
Materialized views — let the DB own the refresh; accept staleness between refreshes.
Async pipelines / CDC / event streams — for cross-service copies, publish a change event and let consumers update their denormalized copy (eventual consistency).

The honest engineering statement: denormalized data is derived data with a consistency SLA. You must decide whether it must be strongly consistent (transactional dual write) or can be eventually consistent (async), and you need a way to rebuild/reconcile it (a batch job that recomputes counters) because it will drift.

Decision rule / summary

Default to a normalized (3NF) schema — it's correct, flexible, and cheap to change. Denormalize only when you have a measured read bottleneck, the read/write ratio is heavily read-skewed, and you have a concrete plan to keep the redundant data consistent. In short: normalize until it hurts, denormalize until it works. Snapshot/historical cases are the one exception where you denormalize up front for correctness, not performance.

#### Common mistakes
/ follow-up probes an interviewer may push on: denormalizing prematurely without profiling; forgetting the write path now has to maintain the copy (leading to stale counts); no reconciliation job; and choosing strong vs. eventual consistency without thinking about the domain (a "like count" can be eventually consistent; an account balance cannot).

**Comparison:**

```
Dimension	Normalized (3NF)	Denormalized
Reads	Slower: multi-table joins / round trips	Faster: fewer joins, single-row lookups
Writes	Simpler: one source of truth	Heavier: must update all copies
Redundancy / storage	Minimal	Higher (duplicated data)
Consistency risk	Low (no anomalies)	Higher — copies can drift; needs sync/reconcile
Best for	OLTP, write-heavy, evolving schemas	Read-heavy hot paths, aggregates, snapshots, OLAP, NoSQL
When to choose	Default	Measured read bottleneck + read-skewed + sync plan
```

**Code:**

```sql
-- 1) Historical snapshot: copy name + price onto the order line at write time.
--    A live join to Products would show TODAY's price -> wrong for an old order.
CREATE TABLE order_lines (
    order_id       BIGINT,
    product_id     BIGINT,
    product_name   TEXT           NOT NULL,   -- denormalized snapshot
    unit_price     NUMERIC(10,2)  NOT NULL,   -- price AT purchase time
    quantity       INT            NOT NULL,
    PRIMARY KEY (order_id, product_id)
);

-- 2) Maintained aggregate: read on every page, kept in sync by a trigger.
ALTER TABLE posts ADD COLUMN like_count INT NOT NULL DEFAULT 0;

CREATE OR REPLACE FUNCTION bump_like_count() RETURNS trigger AS $$
BEGIN
  IF (TG_OP = 'INSERT') THEN
    UPDATE posts SET like_count = like_count + 1 WHERE id = NEW.post_id;
  ELSIF (TG_OP = 'DELETE') THEN
    UPDATE posts SET like_count = like_count - 1 WHERE id = OLD.post_id;
  END IF;
  RETURN NULL;
END; $$ LANGUAGE plpgsql;

CREATE TRIGGER trg_like_count
AFTER INSERT OR DELETE ON likes
FOR EACH ROW EXECUTE FUNCTION bump_like_count();

-- Reconciliation job (run periodically to correct drift):
-- UPDATE posts p SET like_count = c.n
-- FROM (SELECT post_id, COUNT(*) n FROM likes GROUP BY post_id) c
-- WHERE p.id = c.post_id;
```

**Example input / output:**

```text
Normalized read (2 queries / a join per page load):
  SELECT * FROM order_lines WHERE order_id = 42;          -- product_id only
  SELECT name, price FROM products WHERE id IN (...);      -- extra round trip; price is CURRENT

Denormalized read (single query, historically correct):
  SELECT product_name, unit_price, quantity
  FROM order_lines WHERE order_id = 42;
  -> ("Pen", 10.00, 2)  even if the product's price later changed to 12.00

Counter read:
  Normalized:   SELECT COUNT(*) FROM likes WHERE post_id = 7;  -- scans large table each load
  Denormalized: SELECT like_count FROM posts WHERE id = 7;     -- O(1) single-row lookup
```

**Flow:**

1. No
2. Yes
3. Write-heavy
4. Read-heavy
5. Yes
6. No
7. Strong
8. Eventual OK
9. Is this a measured readbottleneck?
10. Stay normalized 3NF
11. Read-heavy vs write ratio?
12. Need point-in-time / historicalvalue?
13. Denormalize: snapshotcolumns
14. Consistency requirement?
15. Transactional dual write /trigger
16. Async CDC / materialized view+ reconcile job
17. Add reconciliation / rebuild job

