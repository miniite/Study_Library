---
type: "practice"
difficulty: "Deep"
---

# Explain the CAP theorem and how it applies to distributed databases.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Define the CAP theorem precisely (not the loose "pick 2 of 3" slogan), explain why the trade-off exists, and show how it drives concrete design choices in real distributed databases (DynamoDB, Cassandra, MongoDB, Spanner, etc.). At "deep understanding" level for a backend/fullstack SDE targeting tier-1/tier-2 companies, the interviewer is probing whether I understand that CAP is a statement about behaviour during a network partition, not a permanent property, and whether I can map it to systems I've actually used and to consistency guarantees like the PACELC extension.

What the interviewer wants to hear:

The formal statement and correct definitions of C, A, P.
The key insight: P is not optional in a real distributed system, so the real choice is C vs A when a partition happens.
Concrete DB examples on each side and the tunable middle ground.
PACELC (the honest, complete framing) and the difference between CAP's linearizability and weaker consistency models.

Plan: Define the three properties → state the theorem and the intuition/proof sketch → clarify the common misconception → classify real databases → introduce PACELC and tunable consistency → summary + common mistakes.

The three properties

#### Cons
istency (C): Every read receives the most recent write or an error. In CAP, "consistency" specifically means linearizability — the system behaves as if there is a single, up-to-date copy of the data and all operations happen in a single global order. (Note: this is a different C from the "C" in ACID, which is about integrity constraints. Same letter, different concept.)
Availability (A): Every request received by a non-failing node returns a (non-error) response — but not necessarily the most recent data. Every live node must answer.
Partition tolerance (P): The system continues to operate despite the network dropping or delaying an arbitrary number of messages between nodes (i.e., the cluster splits into groups that can't talk to each other).
The theorem

CAP theorem (Brewer's conjecture, proven by Gilbert & Lynch, 2002): In the presence of a network partition, a distributed system can guarantee at most one of Consistency or Availability — you cannot have both.

Intuition / proof sketch. Imagine two replicas, N1 and N2, holding value x = 0. A partition cuts the link between them. A client writes x = 1 to N1. Now a second client reads x from N2, which has no way to learn about the new write.

If N2 answers with the stale x = 0, the system is Available but not Consistent.
If N2 refuses to answer (or blocks) until it can confirm it has the latest value, the system is Consistent but not Available.

There is no third option: while the partition exists, N2 must either serve possibly-stale data or stop serving. That's the whole theorem.

The critical misconception to correct

The popular "pick 2 of 3" framing is misleading. Network partitions are a fact of life — cables fail, switches reboot, packets drop, GC pauses look like partitions. You cannot choose to not tolerate partitions in any system that spans multiple machines; a system that isn't partition-tolerant simply stops entirely when the network hiccups, which is the worst outcome.

So the real, practical statement is: When a partition occurs, you must choose between C and A. When there is no partition (the normal case), a well-designed system can offer both strong consistency and high availability. This is why "CA" databases in the classic Venn diagram are essentially single-node systems (a traditional single-instance RDBMS) that don't face the distributed problem at all.

How it applies to real distributed databases

CP systems — choose consistency, sacrifice availability during a partition. The minority side of a partition rejects writes/reads rather than serve stale data.

HBase, MongoDB (with strong write concern / majority reads), etcd, ZooKeeper, Google Spanner, CockroachDB. These use consensus (Paxos/Raft) requiring a quorum/majority. If a replica set can't reach a majority, it steps down and stops accepting writes — you lose availability on the minority partition but never read stale data.
Use when correctness is non-negotiable: config/coordination stores, financial ledgers, inventory decrement.

AP systems — choose availability, sacrifice strong consistency. Every node keeps answering; replicas reconcile later (eventual consistency).

Amazon DynamoDB (in eventually-consistent read mode), Cassandra, Riak, CouchDB. They use replication without requiring a synchronous majority; conflicts are resolved with last-write-wins, vector clocks, or CRDTs.
Use when uptime and low latency matter more than reading the very latest value: shopping carts, social feeds, session stores, telemetry.

Tunable / hybrid. Real systems rarely force one global choice. Cassandra and DynamoDB let you tune consistency per query via quorum settings. Using replication factor N with read consistency R and write consistency W, you get strong consistency when R + W > N (reads and writes overlap on at least one node) and higher availability/lower latency when they don't. So the same cluster can be CP for a critical write (QUORUM) and AP for a cheap read (ONE).

PACELC — the more complete model

CAP only talks about behaviour during a partition. PACELC (Abadi, 2012) extends it: if there is a Partition, choose between Availability and Consistency; Else (normal operation), choose between Latency and Consistency. This captures the everyday reality that even without partitions, synchronous replication for strong consistency costs latency.

Spanner: PC/EC — consistent during partitions and prefers consistency over latency normally (it pays the latency of synchronized clocks/consensus).
DynamoDB / Cassandra (default): PA/EL — available during partitions and favours low latency over consistency in normal operation.

This is the framing that separates a "deep understanding" answer from a textbook one.

Summary — what a strong answer emphasizes
CAP's C = linearizability, A = every live node answers, P = survives network splits.
Partitions are unavoidable, so CAP is really a C-vs-A choice that only bites during a partition; with no partition you can have both.
Map it to systems: CP = Spanner/etcd/ZooKeeper/CockroachDB/HBase; AP = Dynamo/Cassandra/Riak; tunable via quorums (R + W > N).
Cite PACELC to show you know CAP is incomplete (it ignores the latency/consistency trade-off in the common no-partition case).

#### Common mistakes
/ follow-up probes:

Saying "pick any 2" as if all three are symmetric — P isn't optional.
Conflating CAP's "consistency" with ACID's "consistency."
Claiming a system is permanently "CP" or "AP" — modern DBs are tunable per operation.

#### Follow-ups
"Where does your payment service sit and why?" (CP for the ledger, AP for the activity feed), "What consistency does QUORUM/R+W>N give you?", "Is Spanner really CA?" (No — it's CP; it just makes partitions rare via redundant networking, and it pays latency, hence PC/EC).

**Comparison:**

```
Property	CAP meaning	Violated when
Consistency (C)	Linearizable: every read sees latest write	A node returns stale data
Availability (A)	Every non-failing node returns a non-error response	A live node errors/blocks
Partition tolerance (P)	Keeps working despite dropped messages between nodes	System halts entirely on a network split
System	CAP class	PACELC	Mechanism
Spanner / CockroachDB	CP	PC/EC	TrueTime + Paxos/Raft consensus, majority quorum
etcd / ZooKeeper	CP	PC/EC	Raft / ZAB consensus
HBase, MongoDB (majority)	CP	PC/EC	Single primary per shard, majority ack
DynamoDB, Cassandra, Riak	AP (default)	PA/EL	Multi-master replication, tunable quorum
Single-node RDBMS (MySQL one box)	"CA" (no partition to face)	—	Not distributed
Trade-off	CP choice	AP choice
During partition	Reject requests on minority side	Serve possibly-stale data
Good for	Ledgers, inventory, coordination/config	Carts, feeds, sessions, telemetry
Failure symptom	Timeouts / errors, but never wrong	Always answers, but may be stale
```

**Example input / output:**

```text
Quorum consistency worked example (Cassandra / Dynamo-style), replication factor N = 3:

Scenario A — strong consistency:
  Write with W = QUORUM (2 nodes must ack) → key=x, value=1
  Read  with R = QUORUM (2 nodes must respond)
  R + W = 2 + 2 = 4 > N (3)  ⇒  read and write sets overlap on ≥1 node
  Read of x → returns 1  (guaranteed latest)

Scenario B — high availability / low latency, weak consistency:
  Write with W = ONE (1 node acks)
  Read  with R = ONE (1 node responds)
  R + W = 1 + 1 = 2 ≤ N (3)  ⇒  may not overlap
  Read of x → may return 0 (stale) until anti-entropy/read-repair converges

Partition dry-run (2 replicas N1, N2; link cut):
  Client writes x=1 to N1  → N1: x=1, N2: x=0
  Client reads x from N2:
    CP system  → N2 lacks quorum → returns ERROR/blocks  (Consistent, not Available)
    AP system  → N2 returns x=0 (stale)                  (Available, not Consistent)
```

**Flow:**

1. No
2. Yes
3. Choose Consistency CP
4. Choose Availability AP
5. Request arrives at a node
6. Network partition present?
7. Serve with strong consistency+ availabilityPACELC: trade Latency vsConsistency
8. Design choice:C or A?
9. Minority side rejects/blocksnever returns stale data
10. All live nodes answermay return stale, reconcilelater
11. Examples: Spanner, etcd,ZooKeeper, CockroachDB
12. Examples: DynamoDB,Cassandra, Riak

