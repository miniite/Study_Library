---
type: "practice"
difficulty: "Deep"
---

# What is an auto-increment / identity column and what problems can arise with it at scale?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

"What is an auto-increment / identity column and what problems can arise with it at scale?" — a deep-understanding question for backend/fullstack SDEs. The first half is a definition (easy); the real signal the interviewer wants is the second half: can I enumerate the concrete, hard-won failure modes of auto-increment IDs when a system grows — sharding conflicts, hotspots on the index, gaps and their consequences, security/enumeration, replication/DR behavior, and the 32-bit exhaustion outage. A senior answer names each problem, explains why it happens mechanically, and offers the standard mitigations/alternatives (UUIDv7/ULID, Snowflake, sequence blocks, hi/lo). Clarifying context I'd assume: this is an OLTP relational store (Postgres/MySQL) that may later shard or go multi-region.

What an auto-increment / identity column is

An auto-increment (MySQL) / identity (SQL Server, and the SQL-standard GENERATED AS IDENTITY in Postgres) / serial+sequence (classic Postgres) column is a column whose value is automatically assigned by the database as a monotonically increasing integer when you insert a row, so you don't supply it yourself. It's the most common way to generate a surrogate primary key.

Mechanically, the DB keeps a counter (a sequence object, or an internal per-table counter for MySQL AUTO_INCREMENT). On each insert it hands out the next value (n, n+1, n+2, …) and advances the counter. In MySQL InnoDB the value comes from an in-memory auto-increment counter; in Postgres it comes from a SEQUENCE. The guarantees are: unique and, at a single node, roughly ordered by insertion time. That ordering is a genuine benefit — new rows append to the "right end" of the B-tree index, giving excellent write locality.

SQL
-- Postgres (SQL standard, preferred)
CREATE TABLE orders (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  ...
);
-- MySQL
CREATE TABLE orders (
  id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  ...
);
Problems that arise at scale

1. Single point of coordination → breaks under sharding / multi-master. The counter lives in one place. The moment you shard the table across N databases or run multi-master/multi-region writes, two nodes will both mint id=1001 for different rows → primary-key collisions. You can't have two authorities for one monotonic sequence without cross-node coordination on every insert, which is exactly the latency you sharded to avoid. This is the number-one reason large systems abandon plain auto-increment for the PK.

Mitigations: per-shard offset+increment (shard A: 1,3,5…; shard B: 2,4,6…), sequence blocks / hi-lo (each node reserves a range of 1000 IDs at a time), or switch to coordination-free IDs (Snowflake, UUIDv7/ULID).

2. Write hotspot on the index (monotonic insert contention). Because every new row has the largest key, all inserts hit the same rightmost B-tree leaf page and, in MySQL, the same auto-increment mutex/lock. Under very high concurrent insert throughput this becomes a hot page / lock contention point that limits write scalability. Random keys spread inserts across the tree (relieving the hotspot) but wreck locality — so this is a genuine trade-off, and time-ordered IDs like UUIDv7/ULID try to balance it.

3. Information leakage & enumeration (security). Sequential IDs are guessable and countable. Exposing /order/1002 in a URL or API lets an attacker (a) enumerate all resources by incrementing the ID (an IDOR / scraping vector if authz is weak), and (b) infer business volume — sign up twice a day apart, see your user IDs jump by 5,000, and you've learned the daily signup rate (the classic "German tank problem"). Competitors love this.

Mitigations: never rely on obscurity for authz; use opaque/random public IDs (UUIDv7/ULID or a hashed/encoded public id) in external URLs while keeping the int PK internal.

4. Gaps are normal and can mislead. Auto-increment values are not gap-free. A rolled-back transaction, a failed insert, INSERT ... ON CONFLICT, or a server restart can consume/skip values. In MySQL, innodb_autoinc_lock_mode and restarts can leave holes; deletes obviously leave gaps. So treating the ID as "count of rows" or assuming contiguity for pagination/audit is a bug. IDs guarantee uniqueness and rough order, not contiguity.

5. 32-bit exhaustion — a real production outage. An INT/SERIAL (signed 32-bit) maxes out at ~2.147 billion. High-volume tables (events, messages, order-items) hit this, and when they do, every insert fails with a duplicate/overflow error — a full write outage that's painful to fix online (you must ALTER the column to BIGINT, a heavy migration on a huge table). The famous real-world example is systems that ran out of INT IDs and went down.

Mitigation: default to BIGINT (9.2 quintillion) for any table that could grow; the extra 4 bytes/row is cheap insurance.

6. Replication / failover surprises. Auto-increment state must survive failover. On MySQL, the in-memory counter is re-derived on restart (from MAX(id)+1), which can re-issue values that existed only in rolled-back or purged rows, and multi-master needs auto_increment_increment/offset tuned to avoid clashes. Logical replication and blue/green cutovers must carry the sequence's current value or the new primary starts minting duplicates.

7. Cross-service merging / data portability. When you later merge two datasets (two regions, an acquisition, a shard split/rebalance), colliding id=1 rows everywhere make the merge painful. Globally-unique IDs (UUID/ULID) merge trivially.

8. Ordering isn't a reliable timestamp. People assume "higher id = created later," but under concurrency and long transactions, IDs can be assigned in a different order than commit order, so relying on ID for strict chronological ordering (e.g., in a ledger) is subtly wrong — use an explicit timestamp/commit-order column.

Alternatives and when to use each
Auto-increment BIGINT: great default for a single-node OLTP table with internal-only IDs. Best index locality, smallest key. Keep it.
UUIDv4 (random): globally unique, coordination-free, non-enumerable — but destroys index locality (random inserts fragment the B-tree, causing page splits and write amplification), and it's 16 bytes. Fine for low-write or where you never cluster on it; risky as a clustered PK on hot tables.
UUIDv7 / ULID (time-ordered): the modern sweet spot — coordination-free and index-friendly (embeds a millisecond timestamp prefix so inserts still append near the right end) and non-sequential/non-enumerable. Great for sharded and/or public-facing IDs.
Snowflake IDs (64-bit: timestamp + machine-id + per-node sequence): time-sortable, coordination-free across nodes, compact (fits BIGINT). Twitter/Discord-style; needs clock-sync and machine-id assignment.
Sequence blocks / hi-lo: keep auto-increment semantics but let each app node reserve ranges, reducing coordination while staying integer.
Summary — what a strong answer emphasizes
Definition: DB-assigned monotonically increasing integer surrogate PK (sequence/AUTO_INCREMENT/IDENTITY); guarantees uniqueness + rough order + good index locality on one node.
Scale problems (name them): (1) single coordination point → collisions under sharding/multi-master; (2) rightmost-page write hotspot/lock contention; (3) enumeration + volume leakage (security); (4) gaps are normal — never assume contiguity; (5) 32-bit exhaustion outage → always use BIGINT; (6) replication/failover counter hazards; (7) painful cross-dataset merges; (8) ID order ≠ commit order.
Judgment: keep BIGINT auto-increment for single-node internal PKs; move to UUIDv7/ULID or Snowflake when you shard, go multi-region, or expose IDs publicly.

#### Common mistakes
/ follow-up probes: using INT (32-bit) on a big table; exposing sequential IDs externally; assuming no gaps; using UUIDv4 as a clustered PK on a write-hot table; and forgetting to carry the sequence value across failover/replication.

**Comparison:**

```
Problem at scale	Why it happens	Mitigation
PK collisions when sharded/multi-master	One counter can't be the sole authority across nodes	offset+increment, hi-lo blocks, UUIDv7/ULID, Snowflake
Write hotspot on rightmost index page	All inserts get the max key → same leaf/lock	time-ordered but partly distributed IDs; partitioning
Enumeration + volume leakage	IDs are sequential and guessable	opaque public IDs (UUIDv7/ULID); real authz
Unexpected gaps	rollbacks/failures/restarts consume values	don't assume contiguity; count via COUNT(*)
32-bit exhaustion → write outage	INT maxes at ~2.15B	use BIGINT by default
Replication/failover duplicates	counter re-derived / not carried over	carry sequence value; tune increment/offset
Painful dataset merges	colliding low ids everywhere	globally-unique IDs
ID scheme	Coordination-free	Index locality	Enumerable	Size
Auto-increment BIGINT	No	Excellent	Yes	8B
UUIDv4	Yes	Poor (fragments)	No	16B
UUIDv7 / ULID	Yes	Good (time-prefixed)	No	16B
Snowflake	Yes	Good	Partially	8B
```

**Code:**

```

-- The exhaustion trap: INT caps at 2,147,483,647 CREATE TABLE events (id INT AUTO_INCREMENT PRIMARY KEY, payload JSON); -- BAD at scale -- Fix / default: CREATE TABLE events (id BIGINT AUTO_INCREMENT PRIMARY KEY, payload JSON); -- ~9.2e18

-- Sharding mitigation with plain auto-increment (offset + increment): -- shard A mints 1,3,5,... shard B mints 2,4,6,... (no collisions) SET @@auto_increment_increment = 2; -- MySQL, set per node SET @@auto_increment_offset = 1; -- shard A; use 2 on shard B

-- Coordination-free, index-friendly, non-enumerable public IDs (Postgres): CREATE TABLE orders ( id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- internal, fast joins public_id UUID NOT NULL DEFAULT gen_random_uuid_v7() -- exposed in URLs; opaque + time-ordered ); CREATE UNIQUE INDEX ux_orders_public ON orders(public_id);

CODE
Copy

```text
// Snowflake-style 64-bit ID layout (fits in BIGINT), coordination-free:
//  1 bit sign | 41 bits ms-timestamp | 10 bits machine-id | 12 bits per-node sequence
//  -> time-sortable, unique across nodes without a central counter
```

**Example input / output:**

```text
Enumeration / volume-leak attack (the German tank problem):
  Day 1, sign up -> your user id = 40,000
  Day 2, sign up -> your user id = 45,000
  Inference: ~5,000 signups/day. Also GET /users/1..45000 enumerates everyone.
  Mitigation: expose UUIDv7 public_id, keep int id internal.

Gaps are normal (not contiguous):
  INSERT ... (id=101 assigned) COMMIT
  INSERT ... (id=102 assigned) ROLLBACK   <-- 102 consumed, row gone
  INSERT ... (id=103 assigned) COMMIT
  Table now has ids {101, 103}; "102" is a permanent gap. Row count != max(id).

Sharding collision without offset tuning:
  Shard A: INSERT -> id=1001   Shard B: INSERT -> id=1001   -> DUPLICATE PK on merge
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. No
7. Yes
8. No
9. Choosing a PK generationstrategy
10. Single node, internal-only IDs?
11. Auto-increment BIGINTbest locality, smallest key
12. Sharded / multi-region writes?
13. Must fit BIGINT / need maxsortability?
14. Snowflake ID
15. UUIDv7 / ULIDcoordination-free +index-friendly
16. IDs exposed publicly inURLs/APIs?
17. Always BIGINT, never INTto avoid exhaustion outage

