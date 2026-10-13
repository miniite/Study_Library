---
type: "practice"
difficulty: "Intermediate"
---

# What is a primary-replica (leader-follower) database setup and why use it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Explain the primary-replica (also called leader-follower or primary-secondary) database topology — one node accepts writes, one or more nodes copy those writes and serve reads — and the why: read scaling, high availability, backups/analytics offloading, and geo-locality. At "intermediate experience" level they also want the trade-offs: replication lag, read-your-writes, failover, and when it's not enough.

What the interviewer is probing for: Do you know the direction of data flow (writes → primary only, reads → replicas), the difference between synchronous and asynchronous replication, the practical gotcha of replication lag, and how failover/promotion works? Can you say what problem it solves and what it does not solve (write scaling, storage limits)?

#### Assumptions
/ clarifying questions: Is the workload read-heavy (the usual reason to do this)? What's the tolerance for stale reads and for data loss on failover? Same region or cross-region? I'll assume a typical read-heavy web app and call out where the answer changes.

Plan: Describe the topology from first principles → how replication actually works (sync vs async, log shipping) → the concrete reasons to use it → the trade-offs (lag, read-your-writes, failover) → what it does NOT solve → summary + follow-ups.

From first principles

In a primary-replica setup you designate one node as the primary (leader). All writes (INSERT/UPDATE/DELETE, DDL) go to the primary. The primary records every change in a replication log (MySQL binlog, PostgreSQL WAL). Each replica (follower) connects to the primary, streams that log, and replays the same changes in the same order, so it ends up with a copy of the data. Replicas are read-only; the application sends read queries to the replicas and writes to the primary.

That's the whole idea: a single source of truth for writes, with fan-out copies for reads and redundancy. It keeps write logic simple (one ordering authority, no write-write conflicts) while letting you add read capacity by adding more followers.

Synchronous vs asynchronous replication — a key distinction:

Asynchronous (default, most common): the primary commits and acks the client without waiting for replicas. Fast, but replicas lag slightly, and a primary crash can lose the last un-replicated writes.
Synchronous / semi-synchronous: the primary waits for at least one replica to confirm before acking. No data loss on failover, but every write pays the round-trip latency, and if the replica is down the primary may stall. Most systems use semi-sync (wait for one replica) as a middle ground.
Why use it — the concrete reasons
Scale reads. Read-heavy apps (often 90%+ reads) offload SELECTs to N replicas. Ten replicas ≈ 10× read throughput without touching the primary. This is the number-one reason.
High availability / failover. If the primary dies, a replica is promoted to primary and traffic redirects — minutes or seconds of downtime instead of hours restoring from backup. The replica is a warm standby with near-current data.
Offload heavy/analytics work. Run reporting queries, exports, and analytics against a replica so they don't slow down the transactional primary. Take backups from a replica so the primary isn't stalled.
Geo-locality / lower read latency. Place replicas near users (a replica in eu-west serves European reads locally instead of crossing the ocean to the primary).
Zero/low-downtime maintenance. Patch or upgrade a replica, or fail over to it while you maintain the old primary.

#### Trade-offs
and gotchas (what intermediate-level must cover)
Replication lag (staleness). With async replication, a replica may be tens of milliseconds to seconds behind (more under write bursts, big transactions, or network issues). A read right after a write may not see the write — the classic "I updated my profile, refreshed, and it's still the old value" bug.
Read-your-own-writes. To avoid the above: after a user writes, route their reads to the primary for a short window, or route to a replica known to be caught up. This is a session-consistency pattern layered on top.
Failover complexity. Promotion isn't free: you must detect failure (avoid false positives), pick the most up-to-date replica, repoint the app (via a proxy/DNS/virtual IP), and prevent split-brain (two primaries accepting writes). Tools like Patroni, Orchestrator, or managed RDS/Aurora automate this.
Data loss window. Async failover can lose the last un-replicated writes. If that's unacceptable (payments), use semi-sync so at least one replica has every acked write.
Replica consistency model. Reads from replicas are eventually consistent relative to the primary. Fine for feeds/catalogs; not for balance checks that must be exact.
Write bottleneck remains. More replicas do not help writes at all — every write still funnels through the single primary.
What primary-replica does NOT solve
Write scaling — one primary is the write ceiling. To scale writes you need sharding (multiple primaries, each owning a slice). Replication and sharding are complementary: shard for write scale, replicate each shard for read scale + HA.
Storage limits — every replica holds the full dataset, so it doesn't help when the data won't fit one machine. Again, that's sharding's job.
Strong global consistency — replicas are behind unless you read from the primary or use synchronous replication everywhere (which sacrifices latency/availability).
Summary — what a strong answer emphasizes

A primary-replica setup routes all writes to one leader that streams its change log to read-only followers, which replay it and serve reads. You use it to scale reads, gain high availability via failover, offload backups/analytics, and cut read latency geographically — while keeping writes simple with a single ordering authority. The trade-off you must own is replication lag (async is fast but stale; sync is consistent but slow), the read-your-own-writes pattern to hide staleness, and failover discipline (pick the freshest replica, avoid split-brain, accept a small data-loss window unless semi-sync). And be clear about its limit: it scales reads and HA, not writes or storage — that's when you reach for sharding.

#### Common mistakes
thinking replication scales writes (it doesn't); reading from a replica right after writing and being surprised by staleness; assuming failover is automatic and lossless by default; letting two nodes both accept writes (split-brain).

#### Follow-up
probes: "A user updates their email and immediately sees the old one — why, and how do you fix it?" (replication lag → route their reads to primary / caught-up replica). "Primary dies — walk me through failover." (detect → promote freshest replica → redirect via proxy → fence old primary). "You've maxed the primary on writes — now what?" (sharding).

**Comparison:**

```
Replication mode	Client ack waits for	Data loss on primary crash	Write latency
Asynchronous	Primary only	Possible (un-replicated writes)	Lowest
Semi-synchronous	Primary + 1 replica	None (≥1 replica has it)	Medium
Synchronous (all)	Primary + all replicas	None	Highest / can stall
Goal	Primary-replica solves it?	Notes
Scale reads	Yes	Add replicas
High availability / failover	Yes	Promote a replica
Offload analytics/backups	Yes	Query/back up a replica
Geo read latency	Yes	Replica near users
Scale writes	No	Need sharding
Store data too big for one node	No	Need sharding
Exact strong reads everywhere	No	Read primary or use sync repl
```

**Code:**

```javascript
// Application-level read/write splitting with read-your-own-writes handling
const pools = {
  primary: makePool(PRIMARY_URL),          // all writes + fresh reads
  replicas: [makePool(REPLICA1), makePool(REPLICA2)], // scaled reads
};

let rr = 0;
const pickReplica = () => pools.replicas[rr++ % pools.replicas.length];

async function query(sql, params, { write = false, mustBeFresh = false } = {}) {
  // writes -> primary; reads that must reflect a just-made write -> primary too
  const pool = (write || mustBeFresh) ? pools.primary : pickReplica();
  return pool.query(sql, params);
}

// usage
await query('UPDATE users SET email=$1 WHERE id=$2', [email, id], { write: true });
// right after the write, read from primary to avoid replica lag:
await query('SELECT email FROM users WHERE id=$1', [id], { mustBeFresh: true });
// normal browsing read -> a replica:
await query('SELECT * FROM posts ORDER BY created_at DESC LIMIT 20');
```

**Example input / output:**

```

Replication-lag scenario (async):

CODE
Copy
t=0ms   client -> PRIMARY: UPDATE email='new@x.com'  (primary commits, acks)
t=1ms   client -> REPLICA1: SELECT email             -> 'old@x.com'  (STALE, not replayed yet)
t=40ms  PRIMARY streams WAL -> REPLICA1 replays change
t=41ms  client -> REPLICA1: SELECT email             -> 'new@x.com'  (caught up)

Fix: route the immediate post-write read to PRIMARY (mustBeFresh) -> always 'new@x.com'.
```

**Flow:**

1. Writes
2. INSERT/UPDATE/DELETE
3. stream WAL/binlog
4. stream WAL/binlog
5. SELECT
6. SELECT
7. promote on failure
8. Application
9. (Primary / Leader)
10. (Replica 1)
11. (Replica 2)

