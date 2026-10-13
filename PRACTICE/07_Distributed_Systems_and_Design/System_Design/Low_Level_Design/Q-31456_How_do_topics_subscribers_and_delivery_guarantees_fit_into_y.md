---
type: "practice"
difficulty: "Expert"
---

# How do topics subscribers and delivery guarantees fit into your Pub/Sub design?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: A direct follow-up to "design a Pub/Sub broker." Now go deeper on three cross-cutting concerns and how they fit the class model: (1) topics (naming, routing, wildcards, retention), (2) subscribers/subscriptions (fan-out semantics, consumer groups, ordering, offsets vs push), and (3) delivery guarantees — at-most-once, at-least-once, exactly-once — and the concrete mechanisms (ack, retry, idempotency, dedup, DLQ) that make each real, plus the trade-offs.

What the interviewer is probing (expertise level): This is the "do you actually understand messaging semantics" gate. They want to see that you know delivery guarantees are end-to-end contracts, not a config flag; that "exactly-once delivery" is essentially impossible over an unreliable network so we engineer effectively-once via idempotency; that ordering, retries, and guarantees interact (a retry can reorder); and that these choices are per subscription, driving the class design.

#### Assumptions
/ clarifying questions:

Building on the previous class model: Broker → Topic → Subscription → Subscriber, push delivery with per-subscription bounded queue + worker. I'll add acks, offsets, consumer groups, and a durable log where guarantees demand them.
Fault model: subscribers can crash mid-processing; network/acks can be lost; broker can restart. Guarantees must hold under these.

Plan: topics (routing model) → subscribers (fan-out, groups, ordering, push vs pull/offsets) → the three delivery guarantees with exact mechanisms → how each maps onto the classes (what fields/methods appear) → the ordering-vs-retry and guarantee-vs-throughput trade-offs → summary + pitfalls + follow-ups.

1. Topics — the routing abstraction

A topic is the named channel that decouples producers from consumers. Design decisions:

Naming & hierarchy: flat strings (orders.created) or hierarchical with wildcards (orders.*, orders.#). Wildcard matching is done with a topic trie so a publish matches all subscribing patterns efficiently.
Auto-create vs declared: Broker.publish can computeIfAbsent a topic (convenient) or require explicit declaration (safer, avoids typo-topics). Expertise answer: declared topics in production to prevent silent ghost topics.
Retention: transient (push-only, message gone once fanned out) vs durable log (append-only, retained for N hours/GB so late/replaying subscribers can re-read). Durability is a prerequisite for at-least-once across broker restarts.
Partitioning: for scale and ordering keys, a topic splits into partitions by key hash. Ordering is guaranteed within a partition, not across the topic. This is the single most important scale/ordering lever.

So a Topic grows a retentionPolicy, optional partitions[], and (for durable modes) a Log of messages plus per-consumer-group offsets.

2. Subscribers, subscriptions, fan-out
Fan-out (pub/sub) semantics: every distinct subscription to a topic gets every message — this is the copy-per-subscriber model from the base design.

#### Cons
umer groups (queue semantics within pub/sub): a group is a set of subscriber instances that share the load — each message goes to exactly one member of the group. This is how you scale one logical subscriber horizontally. Implementation: partitions of a topic are assigned to group members; each partition → one member (so ordering per key is preserved while load spreads). Subscription therefore belongs to a ConsumerGroup, and the group owns partition→member assignment and the committed offset per partition.
Ordering: guaranteed per partition per group, because a single member drains a partition in FIFO order. Global topic ordering is not offered (it doesn't scale).
Push vs pull / offsets: push (broker calls onMessage) is simple but the broker must track in-flight/backpressure. Pull (Kafka-style: consumer polls and commits offsets) puts flow control in the consumer and makes durability/replay natural. Expertise answer: durable guarantees favor a log + offset model; the Subscription/group holds a committedOffset, and acking = advancing the offset.
3. Delivery guarantees — the heart of the question

Guarantees are end-to-end (broker→subscriber→broker-ack), engineered from acks, retries, persistence, and idempotency. There are three, and the difference is when you ack relative to processing and whether you dedup:

At-most-once (fire-and-forget): broker sends, does not wait for an ack; on failure it does not retry. Zero duplicates, possible loss. Cheapest/fastest. Mechanism: no ack, no persistence, no retry — the base "push and move on." Use for metrics/telemetry where a dropped sample is fine.

At-least-once (the common default): broker delivers, waits for an ack, and retries (with backoff) until acked or DLQ. Persist the message so a broker restart doesn't lose it. No loss, but duplicates possible — because an ack lost on the wire triggers a redelivery of an already-processed message. Consumers must be idempotent (dedup on messageId / dedup key, or make the operation naturally idempotent like an upsert). Mechanism: persist → deliver → await ack → retry → DLQ after N attempts. This is what 90% of systems actually run.

Exactly-once — the honest answer: true exactly-once delivery is impossible over an unreliable network (the Two Generals problem). What we build is effectively-once / exactly-once processing: at-least-once delivery plus consumer-side idempotency/dedup, so duplicates are observed but not acted on twice. Concretely: (a) producer idempotency (dedup identical publishes via a producer sequence number), (b) at-least-once broker delivery, (c) consumer dedup store keyed by messageId (or transactional outbox: write result + mark-processed in one DB transaction). Kafka's "exactly-once" is exactly this: idempotent producer + transactional commits, not magic. Expertise answers name this explicitly and never claim a network delivers exactly once.

How guarantees map onto the classes
Subscriber.onMessage returns/acks: add ack(messageId) / nack(messageId) on the Subscription, or advance committedOffset.
Subscription gains: guarantee (enum), inFlight map (messageId → deliveryDeadline), retryPolicy, maxRetries, deadLetterTopic, and a dedup store reference for idempotent consumers.
Topic gains (durable modes): append-only Log + retention, so redelivery after crash reads from the log by offset.
A redelivery timer requeues messages whose ack deadline passed.

#### Trade-offs
(the judgment part)
Guarantee vs throughput/latency: at-most-once is fastest (no ack round-trip, no persistence); at-least-once adds ack + persistence + dedup cost; effectively-once adds a dedup store lookup/transaction per message. Stronger guarantee ⇒ more coordination ⇒ lower throughput and higher latency.
Ordering vs retries: a naive retry of a failed message while later messages already delivered reorders the stream. To keep per-key order you must either stop-the-partition on failure until the message succeeds/DLQs (blocks head-of-line) or accept reordering. This is a real, forced trade-off (head-of-line blocking vs strict order).
Duplicates vs loss: you cannot have "no loss AND no duplicates AND no coordination." At-least-once trades duplicates for no-loss; at-most-once trades loss for no-duplicates; effectively-once pays coordination cost to hide the duplicates.
DLQ: a poison message must not block the partition forever — after N retries, route to a dead-letter topic for out-of-band handling, and continue.
Summary — what a strong answer emphasizes

Topics are the routing/decoupling and (when durable + partitioned) the ordering and retention unit; subscribers fan out per-subscription, and consumer groups + partitions give horizontal scale with per-key ordering via committed offsets. Delivery guarantees are per-subscription, end-to-end contracts built from acks, retries, persistence, and idempotency: at-most-once (no ack, may lose), at-least-once (ack+retry+persist, may duplicate → needs idempotent consumers), and effectively-once (at-least-once + consumer dedup / transactional processing — because true exactly-once delivery over a network is impossible). The class model absorbs this via Subscription.{guarantee, ack/nack, inFlight, retryPolicy, deadLetterTopic}, a durable Topic.Log with offsets, and a redelivery timer. The senior signal is naming the exactly-once impossibility, the ordering-vs-retry and duplicate-vs-loss trade-offs, and mapping each guarantee to concrete mechanisms.

#### Common mistakes
claiming a broker gives exactly-once "delivery"; forgetting that at-least-once requires idempotent consumers; retrying without a DLQ (infinite poison-message loop); assuming global topic ordering (it doesn't scale — order is per partition); acking before processing (turns at-least-once into at-most-once and silently loses on crash).

Likely follow-ups: how does the ack/offset commit interact with consumer crashes (commit after processing, not before); how do you dedup at scale (bounded TTL dedup store / bloom filter + exact backstop); how do you preserve order under rebalancing (partition reassignment with fencing).

**Comparison:**

```
Guarantee	Mechanism	Loss?	Duplicates?	Consumer requirement	Use case
At-most-once	send, no ack, no retry	Yes (possible)	No	none	metrics, telemetry, heartbeats
At-least-once	persist → send → ack → retry → DLQ	No	Yes (possible)	idempotent	most business events, orders
Effectively-once	at-least-once + consumer dedup / txn outbox	No	Observed but not acted on twice	dedup store / transactional processing	payments, financial ledgers
Concern	Choice	Trade-off
Ordering scope	per partition per group	global order doesn't scale (single-writer bottleneck)
Retry vs order	block partition vs skip-to-DLQ	head-of-line blocking vs reordering
Ack timing	after processing	before-processing silently loses on crash
Scale-out	consumer group + partitions	one member per partition preserves per-key order
Poison msg	DLQ after N retries	avoids infinite redelivery loop
Stronger guarantee	more acks/persistence/dedup	lower throughput, higher latency
```

**Code:**

```java
enum Guarantee { AT_MOST_ONCE, AT_LEAST_ONCE, EFFECTIVELY_ONCE }

// Subscription extended for guarantees, acks, retries, offsets
class Subscription {
    final Guarantee guarantee;
    final int maxRetries;
    final String deadLetterTopic;
    private final Map<String, Long> inFlight = new ConcurrentHashMap<>(); // msgId -> ackDeadline
    private final DedupStore dedup;      // for EFFECTIVELY_ONCE (consumer idempotency)
    private volatile long committedOffset = -1;

    void deliver(Message m, Subscriber s) {
        switch (guarantee) {
            case AT_MOST_ONCE:                    // fire and forget: no ack, no retry
                trySend(m, s); return;            // loss possible, zero duplicates
            case AT_LEAST_ONCE:                   // persist -> send -> await ack -> retry -> DLQ
            case EFFECTIVELY_ONCE:
                for (int attempt = 0; attempt <= maxRetries; attempt++) {
                    if (guarantee == Guarantee.EFFECTIVELY_ONCE && dedup.seen(m.id))
                        { ack(m.id); return; }    // duplicate observed, NOT reprocessed
                    inFlight.put(m.id, System.currentTimeMillis() + 30_000);
                    try {
                        s.onMessage(m);           // process
                        if (guarantee == Guarantee.EFFECTIVELY_ONCE) dedup.record(m.id);
                        ack(m.id); return;        // ack AFTER processing (never before)
                    } catch (Exception ex) {
                        backoff(attempt);         // lost ack / failure -> retry (may duplicate)
                    }
                }
                sendToDeadLetter(m);              // poison message -> DLQ, unblock partition
        }
    }

    void ack(String msgId) { inFlight.remove(msgId); /* advance committedOffset for pull/log */ }
    void nack(String msgId) { /* immediate redelivery */ }
    private void trySend(Message m, Subscriber s) { try { s.onMessage(m); } catch (Exception ignored) {} }
    private void backoff(int a) { try { Thread.sleep((1L << a) * 50); } catch (InterruptedException e) { Thread.currentThread().interrupt(); } }
    private void sendToDeadLetter(Message m) { /* publish to deadLetterTopic */ }
}

// Consumer-side dedup = the mechanism that makes "effectively-once" real
interface DedupStore {                 // bounded TTL store or transactional 'processed' table
    boolean seen(String messageId);
    void record(String messageId);
}

// Redelivery timer for at-least-once: requeue un-acked past deadline
class RedeliveryScheduler {
    void tick(Subscription sub, Map<String, Long> inFlight, DurableLog log) {
        long now = System.currentTimeMillis();
        inFlight.forEach((id, deadline) -> {
            if (now > deadline) sub.nack(id); // redeliver from durable log
        });
    }
}

// Consumer group: share partitions across members, one member per partition (order-safe scale-out)
class ConsumerGroup {
    final String groupId;
    private final Map<Integer, Subscriber> partitionAssignment = new ConcurrentHashMap<>();
    private final Map<Integer, Long> committedOffsets = new ConcurrentHashMap<>();
    void assign(int partition, Subscriber member) { partitionAssignment.put(partition, member); }
    void commit(int partition, long offset) { committedOffsets.put(partition, offset); }
}
```

**Example input / output:**

```text
Scenario A — at-least-once, ack lost on the wire:
  deliver(m1) -> consumer processes -> sends ack -> ACK LOST
  redelivery timer fires -> deliver(m1) AGAIN -> consumer processes m1 twice
  RESULT: no loss, DUPLICATE. (Fine only if consumer is idempotent.)

Scenario B — effectively-once (same failure, with dedup):
  deliver(m1) -> dedup.seen(m1)? no -> process -> dedup.record(m1) -> ack -> ACK LOST
  redelivery -> deliver(m1) -> dedup.seen(m1)? YES -> ack, SKIP processing
  RESULT: no loss, no duplicate side effect. (Observed-once processing.)

Scenario C — at-most-once, consumer crashes mid-process:
  deliver(m1) -> no ack tracked -> consumer crashes before finishing
  RESULT: m1 LOST, never retried. (Acceptable only for telemetry-grade data.)

Scenario D — poison message:
  deliver(m2) -> throws x5 (maxRetries) -> route m2 to orders.created.DLQ
  RESULT: partition unblocked, other messages flow; m2 handled out-of-band.

Ordering vs retry:
  partition p0 = [m1, m2, m3]; m1 fails.
   - strict order: block p0 until m1 succeeds/DLQs (head-of-line blocking)
   - throughput mode: skip m1 to DLQ, deliver m2,m3 (reordering accepted)
```

**Data:**

```yaml
# Guarantee is a per-subscription contract, not a global switch
topic:
  name: orders.created
  partitions: 6                 # ordering unit; key -> hash(key) % 6
  retention: { mode: durable, hours: 72 }   # durable log enables replay/redelivery

subscription:
  group: fulfillment-svc        # consumer group => load shared across 3 members
  guarantee: EFFECTIVELY_ONCE   # AT_MOST_ONCE | AT_LEAST_ONCE | EFFECTIVELY_ONCE
  ack: after-processing         # NEVER before-processing
  maxRetries: 5
  retryBackoff: exponential
  deadLetterTopic: orders.created.DLQ
  dedupStore: { type: redis, ttl: 24h }     # consumer idempotency backstop
```

**Flow:**

1. publish key,payload
2. 1 member per partition
3. at-most-once
4. at-least-once
5. effectively-once
6. ack lost
7. N failures
8. commit offset in txn
9. Publisher
10. Topicpartition = hash key
11. (Partition 0 log)
12. (Partition 1 log)
13. Group member A
14. Group member B
15. Guarantee?
16. send, no ack
17. send -> await ack -> retry ->DLQ
18. dedup.seen? skip :process+record -> ack
19. Redelivery timer requeues ->DUPLICATE
20. (Dead-letter topic)
21. processed once

