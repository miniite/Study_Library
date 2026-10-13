---
type: "practice"
difficulty: "Deep"
---

# What is the CAP theorem in simple terms?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: "What is the CAP theorem in simple terms?" The phrase "in simple terms" plus the deep understanding level is a deliberate trap-and-test: the interviewer wants you to explain it plainly and demonstrate that you understand it precisely — including the subtleties most people get wrong. They are probing whether you know that CAP is about behavior during a network partition, not a general "pick 2 of 3" you decide once at design time. A senior answer nails the intuition, kills the common misconceptions, connects it to real databases and tunable consistency, and mentions PACELC as the more complete model.

Plan: (1) plain-English statement, (2) define C, A, P precisely, (3) the core insight — the choice only bites during a partition, (4) why "CA" is a myth in distributed systems, (5) CP vs AP with real systems and a concrete example, (6) tunable consistency and consistency-vs-availability as a per-operation dial, (7) PACELC and the latency reality, (8) how to reason about it in a design.

Simple statement

The CAP theorem says: in a distributed data store, when a network partition happens (nodes can't talk to each other), you must choose between Consistency and Availability — you cannot have both at that moment. When the network is healthy, you can have both; CAP only forces a choice while the network is broken.

The three properties, precisely

#### Cons
istency (C) — here it means linearizability: every read sees the most recent write. All nodes agree on a single, up-to-date value. (Note: this is not the "C" in ACID — same word, different concept.)
Availability (A) — every request to a non-failed node gets a (non-error, non-timeout) response, even if it might be stale.
Partition tolerance (P) — the system keeps operating even when messages between nodes are dropped or delayed by the network.
The core insight most people miss

CAP is usually phrased "pick 2 of 3," which is misleading. In any real distributed system, network partitions will happen — cables fail, packets drop, a data-center link goes down. So P is not optional; you must tolerate partitions. That means the real decision is a binary one that only matters during a partition: C or A.

Picture two replicas, N1 and N2, and the link between them dies. A client writes x=2 to N1. Now a client reads x from N2, which still thinks x=1. You have two choices:

Return the stale value (or serve the read/write anyway) → you kept Availability but broke Consistency (AP).
Refuse to answer (error/block) until the partition heals and you can confirm the latest value → you kept Consistency but broke Availability (CP).

There is no third option that gives both while the link is down — that impossibility is the theorem.

Why "CA" is essentially a myth for distributed systems

People label single-node relational databases "CA." That's a category error: a single node isn't a distributed system, so partitions between nodes aren't a concern — it's just "not P." The moment you replicate across machines and a partition occurs, a system that dropped P has nothing sensible to do. So for genuinely distributed stores, the meaningful choice is CP vs AP, not a free pick among three.

CP vs AP with real systems
CP (consistency over availability): during a partition, the minority side stops serving to avoid divergence. Examples: HBase, MongoDB (default, primary-based), etcd/ZooKeeper, Google Spanner, and consensus systems (Raft/Paxos). You pick CP for money, inventory, locks, leader election — anywhere a stale/split answer is worse than a temporary error.
AP (availability over consistency): every node keeps answering with possibly stale data and reconciles later (last-write-wins, vector clocks, CRDTs). Examples: Cassandra, DynamoDB, Riak, Couchbase (in their available modes). You pick AP for shopping carts, social feeds, product views, telemetry — where being up matters more than being perfectly fresh, and eventual consistency (replicas converge once the partition heals) is acceptable.

Classic illustration: Amazon's cart is AP — always let people add to cart even during a partition; reconcile conflicts later — because a rejected add costs a sale, while a briefly stale cart does not.

It's a dial, not a one-time decision

Modern systems make C-vs-A tunable per operation. Dynamo-style stores use quorums: with N replicas, if R + W > N (read quorum + write quorum overlap), you get strong consistency; relax them for lower latency and higher availability. DynamoDB offers strongly-consistent vs eventually-consistent reads on the same table; Cassandra sets consistency level per query (ONE, QUORUM, ALL). So "is it CP or AP?" is often answered "it depends which operation and how we configured it."

PACELC — the more honest model

CAP only talks about the partition case and ignores everyday cost. PACELC extends it: if Partition (P) → choose Availability or Consistency (A/C); Else (E, normal operation) → choose Latency or Consistency (L/C). This captures the reality that even with no partition, strong consistency costs latency (extra round-trips/quorum), so you constantly trade freshness against speed. Under PACELC: DynamoDB/Cassandra are PA/EL (favor availability then latency); Spanner is PC/EC (consistency in both cases, paying latency, made viable by TrueTime); MongoDB is roughly PC/EC by default.

How I reason about it in a design

I don't ask "which two do I want." I ask: when the network splits, what should each operation do? For a write to a ledger or a distributed lock → block/error (CP): a wrong or duplicated answer is unacceptable. For a "like," a view count, or adding to a cart → keep serving stale and reconcile (AP): availability wins and eventual consistency is fine. Often the same product uses both — CP for payments, AP for the feed — which is why CAP is a per-service, even per-operation, judgment call.

Summary — what a strong answer emphasizes

CAP = during a network partition, a distributed system must sacrifice either strong Consistency or Availability; partitions are inevitable so P is mandatory, making the real choice CP vs AP. "CA" isn't meaningful for distributed systems. The choice is often tunable per operation (quorums, consistency levels), and PACELC is the more complete lens because it adds the ever-present latency-vs-consistency trade-off during normal operation.

#### Common mistakes
to avoid: treating CAP as a permanent "2 of 3" design choice; conflating CAP's C with ACID's C; claiming a system is "CA"; and forgetting that partitions are rare-but-guaranteed. Follow-up probes: "give an AP and a CP system and why," "how do quorums (R+W>N) give strong consistency," and "explain PACELC / how Spanner gets consistency without sacrificing availability."

**Comparison:**

```
	CP (Consistency + Partition tolerance)	AP (Availability + Partition tolerance)
During a partition	Refuse/block on some nodes to stay correct	Keep serving, may return stale data
Guarantee	Every read sees latest write (linearizable)	Eventual consistency; converge after heal
Cost	Reduced availability, higher latency	Possible temporary inconsistency/conflicts
Good for	Payments, inventory, locks, leader election	Carts, feeds, likes, views, telemetry
Example systems	Spanner, HBase, etcd/ZooKeeper, MongoDB (default)	Cassandra, DynamoDB, Riak, Couchbase

PACELC quick map:

System	If Partition	Else (normal)	Label
DynamoDB / Cassandra	Availability	Latency	PA/EL
Google Spanner	Consistency	Consistency	PC/EC
MongoDB (default)	Consistency	Consistency	PC/EC
```

**Example input / output:**

```

Concrete partition trace (two replicas N1, N2; link down):

CODE
Copy
t0  N1: x=1   N2: x=1        (in sync)
t1  --- network partition: N1 <-X-> N2 ---
t2  client writes x=2 to N1  -> N1: x=2   N2: x=1  (diverged)
t3  client reads x from N2:

    AP choice:  return x=1   (AVAILABLE but STALE  -> consistency broken)
    CP choice:  return ERROR/timeout  (CONSISTENT but UNAVAILABLE)

t4  --- partition heals ---
    AP: replicas reconcile -> both x=2 (eventual consistency; conflicts
        resolved by last-write-wins / vector clocks / CRDTs)
    CP: N2 resumes serving once it confirms latest value x=2

Quorum example (N=3): W=2, R=2 → R+W=4 > 3 → any read overlaps the latest write → strongly consistent. Drop to R=1 → faster, more available, may read stale.
```

**Flow:**

1. No
2. Yes, P is mandatory
3. Keep answering, allow stale
4. Block/error to stay correct
5. Distributed store, replicated
6. Network partition?
7. Both C and A possiblePACELC: still trade Latency vsConsistency
8. Choose behavior
9. AP: AvailableEventual consistencyCassandra, DynamoDB
10. CP: ConsistentReduced availabilitySpanner, etcd, MongoDB
11. On heal: reconcile via LWW /vector clocks / CRDTs
12. On heal: resume once latestvalue confirmed

