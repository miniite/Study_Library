---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/04_Distributed_Failures/Fencing_and_System_Models"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Fencing Tokens and System Models

**L4 Advanced** · [↑ Faults and Failure in Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Because a node can wrongly believe it still holds a lock or leadership, protected resources must verify a monotonically increasing *fencing token* — and every protocol is only as strong as the system model (synchrony and fault type) it assumes.

### 1. The big picture
Failure detectors, leases and timeouts all rest on time, and time in a distributed system lies: a process can be paused for seconds (GC, VM pause, swapping) and then resume believing nothing happened. Its lease has meanwhile expired and another node has been elected. Now two nodes both think they are "the chosen one". This is the classic *zombie leader* problem.

Fencing is the cure at the data layer: instead of trusting the client's belief, make the *resource* reject stale actors. The second half of the lesson widens the lens to **system models** — the explicit assumptions about timing and failures (synchronous, asynchronous, partially synchronous; crash-stop, crash-recovery, Byzantine) that decide which algorithms are even possible. This pairs with Failure Detection, Network Faults and Timeouts and Unreliable Clocks, and leads into leader election and consensus in the Transactions and Consistency block.

### 2. Core concepts and vocabulary
- **Lock / lease** — exclusive right to a resource; a lease is a lock with an expiry so that a crashed holder cannot block others forever.
- **Zombie / stale holder** — a node that no longer holds the lock but doesn't know it.
- **Fencing token** — a number issued with each lock grant that strictly increases; the resource rejects any request carrying a lower token than one it has already seen.
- **Process pause** — a stop of unknown duration (stop-the-world GC, suspension, page faults, SIGSTOP) during which the process cannot notice time passing.
- **Synchronous model** — bounded network delay, bounded pauses, bounded clock error.
- **Asynchronous model** — no timing assumptions at all.
- **Partially synchronous model** — behaves synchronously *most of the time* but occasionally exceeds bounds; the realistic default.
- **Crash-stop / crash-recovery** — a node fails by halting forever / by halting and possibly restarting with durable state intact.
- **Byzantine fault** — a node sends arbitrary, possibly malicious or contradictory messages.
- **BFT (Byzantine fault tolerant)** — continues correctly despite Byzantine nodes; typically needs more than two-thirds honest nodes.
- **Safety vs liveness** — nothing bad ever happens vs something good eventually happens.

### 3. How it works, step by step
**The failure without fencing**
1. Client 1 acquires a lease from the lock service.
2. Client 1 stalls (GC pause) just before writing to storage.
3. The lease expires; the lock service grants it to Client 2.
4. Client 2 writes to storage — correct.
5. Client 1 wakes, still believes it holds the lease, and writes — corrupting the file or overwriting newer data.

**The same sequence with fencing tokens**
1. The lock service returns a token with every grant: Client 1 gets 33.
2. Client 1 pauses; the lease expires; Client 2 gets token 34.
3. Client 2 writes with token 34; the storage service records "highest token seen = 34".
4. Client 1 resumes and writes with token 33; storage sees 33 < 34 and **rejects** it.

![Writes are accepted only in increasing token order](<../../../../_assets/Designing_Data_Intensive_Applications_The_Big_Ideas_Behind_R/ddia_0805.png>)

Key properties:
- The token must come from a **monotonic** source — a counter in the lock service, or ZooKeeper's `zxid` / node version.
- **The resource must check.** Clients checking their own lock status is not enough because the check itself can be delayed past a pause. If the storage can't compare tokens, you can sometimes embed the token in a file name or row version, but *some* server-side check is required.
- Server-side validation is also good hygiene: a service should not assume clients are well-behaved.

**Choosing a system model**
| Axis | Options | What it buys you |
|---|---|---|
| Timing | synchronous / partially synchronous / asynchronous | Whether timeouts and clocks can be trusted for correctness |
| Node faults | crash-stop / crash-recovery / Byzantine | Whether nodes can lie or lose state |

Real data systems design for **partial synchrony + crash-recovery**: they rely on timeouts for *liveness* (progress eventually) but never for *safety* (correctness must hold even if every timeout fires at the wrong moment). Fencing is exactly a safety mechanism that survives wrong timing.

### 4. Worked example
Protecting writes with ZooKeeper-style tokens:

```text
lock-service.acquire("/jobs/nightly")  -> {lease: 30s, token: 33}

client1: write(file, data, token=33)    # pauses 40s before sending
lock-service (lease expired) -> client2: {token: 34}
client2: write(file, data, token=34)    # storage: maxToken = 34, accepted
client1 (wakes): write(file, data, token=33)
storage: 33 < maxToken 34  -> REJECT (stale token)
```

Equivalent in a database: add `fencing_token BIGINT` to the row/metadata and make the update conditional:

```sql
UPDATE job_state
SET    owner = @me, payload = @payload, fencing_token = @token
WHERE  job_id = @job AND fencing_token < @token;   -- 0 rows affected => you are stale
```

Check `@@ROWCOUNT` (or the driver's affected-rows) — zero means a newer holder has already written and you must abort. The same idea shows up as Kafka producer **epochs** (zombie producers are fenced by `transactional.id` epoch), HDFS NameNode fencing (STONITH/`sshfence` before promoting standby), and Raft/Paxos **terms**/ballot numbers.

### 5. Under the hood
- **Why pause-proofing the client is impossible.** Any "check lease, then act" sequence has a window; a pause can land inside it, and the process has no way to observe the pause. The only reliable defense is to let the *resource* arbitrate with a total order.
- **Tokens vs clocks.** Using wall-clock timestamps as tokens is unsafe because clocks drift and jump (see Unreliable Clocks); logical counters avoid that.
- **Leases are still useful** — they bound how long a crashed holder blocks others — but they provide liveness, not safety.
- **Byzantine limits.** Fencing stops *inadvertent* errors; a node that deliberately forges a high token defeats it. Crash-fault protocols (Raft, Paxos, ZAB) assume nodes are honest-but-unreliable. BFT protocols typically tolerate *f* faulty nodes out of 3*f*+1, cost many more messages, and cannot protect against a bug or exploit that hits all replicas identically (same software on every node).
- **Weak forms of lying** are worth guarding against cheaply: application-level checksums for corrupted packets, input range and size validation, and NTP clients that query several servers and discard outliers.
- **Where BFT is justified:** aerospace (radiation bit-flips), and multi-organization systems with mutual distrust such as blockchains. Inside one datacenter, authentication, access control and encryption are the normal defense.

### 6. Performance and design implications
- Token checking adds a comparison and a tiny piece of state per resource — negligible cost compared to the corruption it prevents.
- It forces the resource to be stateful regarding the "highest token seen", which must itself be durable (a restart that forgets it reopens the hole).
- Protocols designed for BFT cost an order of magnitude more messages and don't scale like crash-fault protocols; choose them only when the threat model demands it.
- Design rule: **correctness must not depend on timing assumptions holding**; only progress may.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| ZooKeeper `zxid`, znode `version`/`cversion` | Monotonic numbers usable as fencing tokens | Building a lock or leader service |
| Kafka `transactional.id` + producer epoch | Fences older producer instances | Exactly-once writers after restarts |
| HDFS `dfs.ha.fencing.methods` (`sshfence`, `shell`) | Forces the old active NameNode down before failover | Preventing split-brain in HA |
| Optimistic version column / `ETag` + `If-Match` | Conditional writes that reject stale actors | Application-level fencing |
| Raft term / Paxos ballot | Epoch number every message carries | Understanding why stale leaders are ignored |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fencing tokens with a resource that can validate them | The resource is a black box that can't compare tokens (you must wrap it) |
| Leases for crash recovery of locks | Leases as the *only* protection against concurrent writers |
| Crash-fault-tolerant consensus (Raft, Paxos) | Participants are mutually distrusting organizations |
| BFT protocols | Single-organization datacenter where cost outweighs benefit |
| Partially synchronous model for design | Hard real-time systems with true bounded latency guarantees (different tooling) |

### 9. Common mistakes
- "Check `lock.isValid()` then write" → the pause lands between check and write → data corruption → validate a token at the resource.
- Using timestamps instead of counters as tokens → clock skew lets an old holder win → monotonic logical counter.
- Not persisting the highest token seen → restart accepts a stale writer → store it durably with the data.
- Assuming a deliberately malicious node is covered → forged tokens → use authentication/BFT only where the threat is real.
- Treating GC pauses as impossible because "ours are short" → rare multi-second pauses cause rare, hard-to-reproduce corruption.

### 10. 🎯 Interview kit
**30-second answer:** A paused process can come back thinking it still holds a lock. Fencing tokens fix that: the lock service hands out an increasing number with each grant, and the storage rejects any write with a token lower than one it has already seen, so the zombie is blocked at the resource, not trusted to self-check.
**Structure a longer answer:** describe the pause/lease-expiry race → why client-side checks fail → token mechanism with an example → require server-side enforcement → relate to epochs/terms in Kafka, Raft, HDFS fencing → mention system model and that safety must not rely on timing.
**Likely follow-ups:**
- *Why not just use a short lease and good clocks?* → pauses and clock error can always exceed any bound.
- *What if the storage can't check tokens?* → wrap it, or encode the token in the object name/version.
- *Does fencing stop malicious nodes?* → no; that's Byzantine territory.
- *What system model do real databases assume?* → partial synchrony, crash-recovery, honest nodes.
**Signals interviewers look for:** you separate safety from liveness, know that the resource must enforce the check, and can name real fencing mechanisms (epochs, terms, STONITH).

### 11. Check yourself
- [ ] Draw the Client 1 / Client 2 / storage timeline with and without tokens.
- [ ] Explain why a client-side lease check can never be sufficient.
- [ ] Name the system model most data systems assume and what each assumption provides.
- [ ] Explain what Byzantine faults are and why most data infrastructure ignores them.
- [ ] Give two real systems that use epochs/terms as fencing tokens.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › Fencing tokens
- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › Byzantine Faults
- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › Weak forms of lying

