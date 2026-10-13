---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Event_Streams_and_Messaging"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Event Streams and Messaging Systems

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An event stream is delivered from producers to consumers by a messaging system, and what it does when consumers are slow or crash — drop, buffer, apply backpressure; lose or keep messages — defines which system fits which job.

### 1. The big picture

In batch processing a file is written once and read by many jobs. The streaming equivalent: an **event** (a small, immutable, timestamped record of something that happened) is generated once by a **producer** and processed by one or more **consumers**. Related events are grouped into a **topic** or **stream**, just as related records live in a file.

Consumers could poll a database for new events, but the more often you poll the smaller the fraction of polls that find anything, so overhead rises. Better: notify consumers when events appear. Databases traditionally do this poorly (triggers are limited), so dedicated **messaging systems** emerged. This lesson covers what they are, how they differ, and the delivery behaviours (load balancing, fan-out, acknowledgements, redelivery, reordering) you must understand to use them.

### 2. Core concepts and vocabulary

- **Messaging system / publish-subscribe** — producers publish messages to a topic; the system delivers them to subscribers. Unlike a Unix pipe or TCP connection (one sender, one receiver), many producers and many consumers can share a topic.
- **Message broker (message queue)** — a server, essentially a database optimized for message streams. Producers and consumers are clients.
- **Direct messaging** — producers talk straight to consumers without an intermediary (UDP multicast, ZeroMQ, webhooks, StatsD).
- **Backpressure / flow control** — blocking the producer when the consumer cannot keep up.
- **Buffering / queueing** — holding messages until consumers are ready.
- **Load balancing (shared subscription)** — each message goes to *one* consumer of a group, to share the work.
- **Fan-out** — each message goes to *all* subscribers/groups.
- **Acknowledgement (ack)** — consumer tells the broker "done"; only then is the message removed.
- **Redelivery** — an unacked message is sent again, possibly to a different consumer.
- **Message queue vs streaming platform** — a queue deletes a message after delivery; a streaming platform keeps an ordered, replayable log of events.
- **Dead-letter queue** — where messages that repeatedly fail are parked.

### 3. How it works, step by step

Two questions differentiate every messaging system:

**Q1. What if producers send faster than consumers can process?**

| Option | How it works | Example |
|---|---|---|
| Drop | Excess messages discarded | Best-effort metrics over UDP |
| Buffer | Queue grows (memory, then disk) | Most brokers |
| Backpressure | Producer blocked when buffer is full | Unix pipes, TCP |

If buffering, ask: what happens when the queue no longer fits in memory — crash, or spill to disk (and how slow is that)?

**Q2. What if nodes crash or go offline — are messages lost?** Durability needs disk writes and/or replication, which cost throughput and latency. Occasional loss is fine for periodic sensor metrics, not for counting events or payments.

**Direct messaging** works well in its niche (low-latency market data via UDP multicast; webhooks via HTTP callbacks) but the application must handle loss: if a consumer is offline it misses messages, and if the producer crashes its retry buffer is lost.

**Broker-mediated messaging**:

1. Producer sends a message to a named queue/topic and waits only for the broker's confirmation that it buffered it.
2. Broker stores it (memory and/or disk, possibly replicated).
3. Broker delivers it to consumers *asynchronously* — often within milliseconds, but arbitrarily later if there is a backlog.
4. The consumer processes it and **acks**.
5. The broker deletes the message.
6. If no ack arrives (connection closed or timeout), the broker assumes failure and **redelivers**.

**Multiple consumers:**

```text
(a) Load balancing                  (b) Fan-out
   producer ─► [topic] ─┬─► C1          producer ─► [topic] ─┬─► C1 (gets all)
                        └─► C2                              └─► C2 (gets all)
   each message → one consumer           each message → every consumer
```

They combine: two consumer groups each receive every message (fan-out between groups), while inside each group a message goes to one member (load balancing).

### 4. Worked example

An order service publishes `OrderPlaced` events. Two systems care: *Billing* (must process every order once) and *Analytics* (reads everything for dashboards).

```text
orders ──► topic "orders"
            ├── group "billing"   : 3 workers share messages (load balancing)
            └── group "analytics" : 1 consumer gets all messages (fan-out)
```

Failure walk-through for the billing group (messages m1…m5 sent in order):

1. Worker 1 receives m1, m4; worker 2 receives m2, m3, m5 (arbitrary assignment).
2. Worker 2 crashes while handling m3. No ack arrives.
3. The broker redelivers m3 — to worker 1, which has meanwhile processed m4.
4. Worker 1 now handles **m4, then m3**: the order differs from the sending order.

Consequences and fixes:

- If events are independent (two different customers' orders), reordering is harmless.
- If m3 and m4 are causally related (e.g., "create account" then "change address"), reordering corrupts results. Use a separate queue per consumer (no load balancing), or route all messages with the same key to the same consumer (partitioning).
- The ack may be lost even when processing succeeded → the message is processed twice → consumers must be **idempotent**, or processing and ack must be made atomic.

### 5. Under the hood

- **Brokers are databases with different defaults.** They typically delete a message once acked, assume a small working set (short queues), and notify clients when data changes, rather than supporting arbitrary queries. Performance degrades when queues grow beyond memory.
- **Queue capacity is not infinite.** Enlarging a queue does not raise the processing rate — it only adds latency. Backpressure must be designed in, not added later. Process-local queues serve decoupling, pipelining and burst absorption; size them by measured processing time and acceptable latency.
- **Delivery semantics.** *At-most-once*: send and forget (loss possible). *At-least-once*: ack + redelivery (duplicates possible). *Exactly-once*: at-least-once plus idempotence or atomic commit of effect and ack.
- **Reordering is intrinsic** to load balancing plus redelivery, even if the broker "preserves order" per the JMS/AMQP standards.
- **Messages are opaque bytes plus metadata.** Brokers enforce no data model; use backward/forward-compatible encodings (Avro, Protobuf) so producers and consumers evolve independently. A consumer that republishes must preserve unknown fields.
- **Queue vs streaming platform.** In a queue, a message is raw data passed between systems and then gone. A streaming platform (log-based) keeps an ordered, append-only record that consumers read by offset, so new consumers can replay history and each consumer's progress is just a number. This is why logs suit data pipelines and CDC (see *Partitioned Logs*).
- **Request/response.** A topic is one-way, but a consumer can publish to a reply queue read by the original sender, approximating RPC asynchronously.

### 6. Performance and design implications

- Throughput is traded against durability: synchronous disk writes and replication slow producers; unreliable direct messaging is fastest.
- Per-message acknowledgements add round trips — batch acks or prefetch (consumer window) to raise throughput.
- Unbounded queues hide overload until memory/disk exhaustion; watch queue depth and age of the oldest message.
- Load balancing sacrifices ordering; partition by key where order matters.
- Redeliveries and poison messages can create loops — cap retries and use a dead-letter queue.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Consumer group / shared subscription | Load-balances messages over members | Scaling expensive processing |
| Topic subscription / exchange binding | Fan-out to every subscriber | Several independent consumers |
| Prefetch / QoS | Max unacked messages per consumer | Controlling throughput and fairness |
| Ack mode (auto vs manual) | When a message is considered done | Avoid loss on consumer crash |
| Visibility / ack timeout | Time before redelivery | Tuning for slow handlers |
| Dead-letter queue / max retries | Parks repeatedly failing messages | Poison-message handling |
| Persistent vs transient delivery | Disk-backed or memory-only | Durability vs speed |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Broker + ack/redelivery** for reliable task handoff | Ordering between related messages is essential and you load-balance |
| **Direct messaging** (UDP, ZeroMQ) for lowest latency | Consumers may be offline or loss is unacceptable |
| **Backpressure** when you can slow the source | The producer cannot be blocked (user-facing, sensors) |
| **Buffering/queueing** to absorb bursts | Sustained overload — queue grows forever |
| **Log-based streaming platform** for replay and many consumers | You need per-message priority, delay or complex routing (classic brokers) |

### 9. Common mistakes

- **Assuming messages arrive in order with multiple consumers** → reordering after redelivery → partition by key or one queue per consumer.
- **Auto-ack before processing** → crash loses the message → ack after successful processing.
- **Non-idempotent handlers with at-least-once delivery** → duplicate side effects → dedupe or upsert.
- **Bigger queue as a fix for slow consumers** → only more latency → scale consumers or apply backpressure.
- **Using UDP metrics for counters** → silently wrong totals → use reliable transport for counts.
- **No dead-letter handling** → one bad message blocks or loops forever.

### 10. 🎯 Interview kit

**30-second answer:** A messaging system moves events from producers to consumers. The design questions are what happens when consumers are slower (drop, buffer or backpressure) and what happens on crashes (durability). Brokers buffer messages, deliver asynchronously, and use acks and redelivery for reliability, with load balancing to share work and fan-out to broadcast.

**Structure a longer answer:** clarify requirements (loss, ordering, latency) → direct vs broker → delivery patterns (load balancing, fan-out) → ack/redelivery and its consequences (duplicates, reordering) → queue vs log trade-off → scenario.

**Likely follow-ups:**
- *Why can messages be reordered even in an ordered queue?* → Redelivery of an unacked message to another consumer under load balancing.
- *How do you get effectively-once processing?* → At-least-once plus idempotent consumers or transactional commit.
- *Queue vs Kafka-style log?* → Queue deletes after ack and has per-message state; log retains and consumers track offsets.
- *When use direct messaging?* → Low-latency, loss-tolerant, always-online parties.
- *What is backpressure?* → Blocking producers instead of unbounded buffering.

**Signals interviewers look for:** knows the loss/overload trade-offs, understands ack/redelivery side effects, doesn't claim exactly-once for free.

### 11. Check yourself

- [ ] Name the three options when producers outpace consumers and give an example of each.
- [ ] Draw load balancing and fan-out and show how to combine them.
- [ ] Walk through the m3/m4 redelivery reordering example.
- [ ] Explain why the ack can be lost even if processing succeeded and what that implies for handlers.
- [ ] Contrast a message queue with a streaming platform.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Messages and Streams
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Messaging Systems
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Message Queues and Event-Streaming Platforms
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Scalability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Topics
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Multiple consumers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Direct messaging from producers to consumers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Message brokers
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Consumer Pull and Push
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Message queues

