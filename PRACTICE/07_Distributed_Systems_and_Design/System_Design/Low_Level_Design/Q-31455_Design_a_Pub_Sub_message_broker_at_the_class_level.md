---
type: "practice"
difficulty: "Deep"
---

# Design a Pub/Sub message broker at the class level.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Produce a low-level (class-level) design of an in-process (or single-node) Pub/Sub message broker — the classes, their responsibilities, relationships, and the core method signatures — not a distributed-systems whiteboard of Kafka clusters. The interviewer wants clean OO decomposition, correct concurrency, and extensibility.

What the interviewer is probing:

Can you identify the right entities (Broker, Topic, Subscription/Subscriber, Message, Publisher) and give each a single responsibility?
Do you understand decoupling — publishers don't know subscribers, mediated by a topic?
Concurrency correctness: many publishers and subscribers hitting shared topic state concurrently. Do you reach for concurrent collections, avoid blocking the publisher on slow subscribers (async dispatch), and handle backpressure?
Extensibility hooks: sync vs async delivery, per-subscriber queues, at-least-once vs at-most-once, filtering.

#### Assumptions
/ clarifying questions I'd state up front:

In-process broker (single JVM) — I'll design that and note how it generalizes.
Push delivery model (broker pushes to subscriber callbacks) for the core; I'll mention pull.
Delivery guarantee target: at-least-once with per-subscriber queues; degrade gracefully to at-most-once/best-effort is possible. (Full guarantees are the next question.)
Multiple topics; a subscriber can subscribe to many topics; ordering preserved per topic per subscriber.

Plan: enumerate entities and responsibilities → define interfaces and classes with key methods → design the concurrency model (thread-safe registry, async dispatch, per-subscriber queue, backpressure) → walk a publish flow → note extensibility (filters, DLQ, guarantees) → summary + pitfalls.

Entities and responsibilities
Message<T> — immutable envelope: id, topic, payload, timestamp, optional headers/key. Immutability makes it safe to share across threads/subscribers.
Publisher — thin client that calls broker.publish(topic, payload). Often just the broker's publish API; a dedicated class is optional.
Subscriber (interface) — the consumer contract: onMessage(Message m). Also onError(...) for delivery/handler failures.
Subscription — the binding of one subscriber to one topic. Holds the subscriber ref, an optional filter/predicate, its own bounded queue, a dispatch worker, and delivery bookkeeping (ack, retry count). This is where per-subscriber isolation lives — a slow subscriber only fills its own queue.
Topic — a named channel. Owns its set of Subscriptions and offers subscribe()/unsubscribe()/publish(). Fans a message out to each subscription.
Broker — the façade/registry and lifecycle owner. Maintains the topicName → Topic map (auto-creates topics), exposes publish/subscribe/unsubscribe, owns the thread pools, and handles shutdown. This is the mediator that fully decouples publishers from subscribers.

Relationships: Broker 1—* Topic; Topic 1—* Subscription; Subscription 1—1 Subscriber. A Subscriber can appear in many Subscriptions (across topics).

Concurrency model — the crux

A correct broker must survive many concurrent publishers and slow subscribers:

Thread-safe registries. Broker.topics is a ConcurrentHashMap; Topic.subscriptions is a CopyOnWriteArraySet (subscribe/unsubscribe are rare; publish/iterate are hot — copy-on-write gives lock-free reads).

Don't run subscriber code on the publisher's thread. If publish() synchronously invoked every onMessage, one slow/blocking subscriber would stall the publisher and all others. Instead, publish() enqueues the message into each subscription's bounded queue (ArrayBlockingQueue) and returns fast. A per-subscription dispatch worker (or a shared thread pool with a dedicated worker task per subscription) drains the queue and calls onMessage. This gives isolation (slow subscriber ≠ everyone's problem) and ordering (single worker per subscription drains FIFO → per-topic order preserved for that subscriber).

Backpressure. When a subscription's bounded queue is full you must choose a policy: block the publisher (flow control), drop-oldest/drop-newest (best-effort), or route to a dead-letter queue. Make it a per-subscription strategy.

Failure handling. If onMessage throws, the worker catches it, calls onError, and (for at-least-once) retries with backoff up to N times before DLQ — never let a bad handler kill the worker thread.

Publish flow

publish(topic, payload) → broker looks up/creates the Topic → creates immutable Message → topic iterates subscriptions → for each, evaluate filter → if it passes, offer to that subscription's bounded queue (apply backpressure policy). The dispatch worker for each subscription independently pulls and delivers. Publisher latency is ~O(#subscribers enqueue), not O(slowest subscriber).

Extensibility hooks (design for change)
Filtering / content-based routing: Subscription holds a Predicate<Message>; only matching messages are enqueued.
Delivery mode: Subscription carries a policy enum {SYNC, ASYNC} and a guarantee {AT_MOST_ONCE, AT_LEAST_ONCE}. Sync = deliver on publisher thread (rare, for ordering-critical, low-fanout); async = default.
Dead-letter queue & retries: a topic-level or subscription-level DLQ captures poison messages.
Pull model / offsets: for durability, store messages in a per-topic log and let subscribers track an offset (Kafka-style) — mentioned as the durable evolution; my core is push.
Wildcards / hierarchical topics (orders.*) via a topic-matching trie.
Summary — what a strong answer emphasizes

The design centers on a Broker mediator holding a concurrent topicName → Topic map, Topics that fan out to Subscriptions, and Subscriptions that own a bounded per-subscriber queue + dispatch worker so publishers never block on slow consumers, ordering is preserved per topic per subscriber, and failures/backpressure are contained. Immutability of Message makes cross-thread sharing safe; concurrent/copy-on-write collections make the hot path lock-free. The classes have single responsibilities and clean extension points (filters, delivery modes, DLQ, guarantees). A senior answer explicitly separates the decoupling concern (mediator) from the delivery concern (per-subscription async worker + backpressure).

#### Common mistakes
(1) calling subscriber callbacks synchronously on the publisher thread — one slow consumer stalls the system; (2) a single shared queue for all subscribers — breaks isolation and per-subscriber ordering; (3) ignoring backpressure so an unbounded queue OOMs the broker; (4) letting a thrown handler kill the dispatch thread; (5) mutable Message shared across subscribers causing data races.

Likely follow-ups: How do delivery guarantees (at-least-once/exactly-once) and durability fit in (next question), how do you scale this across nodes, and how do you preserve ordering under retries.

**Comparison:**

```
Class	Responsibility	Key methods	Key state
Message	Immutable event envelope	(fields only)	id, topic, payload, headers, timestamp
Subscriber	Consumer contract	onMessage, onError	(user impl)
Subscription	Bind subscriber↔topic; isolate + deliver	enqueue, runLoop, deliver, stop	queue, filter, policy, retries, worker
Topic	Named channel; fan-out	subscribe, unsubscribe, publish	CopyOnWriteArraySet<Subscription>
Broker	Mediator, registry, lifecycle	publish, subscribe, unsubscribe, shutdown	ConcurrentHashMap<String,Topic>, thread pool
Concern	Decision	Why
Registry concurrency	ConcurrentHashMap / CopyOnWriteArraySet	lock-free hot reads, safe rare writes
Delivery thread	Per-subscription worker, not publisher thread	isolation + per-subscriber FIFO ordering
Queue	Bounded ArrayBlockingQueue per subscription	backpressure, no unbounded OOM
Slow consumer	BLOCK / DROP / DLQ policy	contain blast radius
Handler failure	catch + retry w/ backoff + DLQ	worker survives poison messages
```

**Code:**

```java
import java.util.*;
import java.util.concurrent.*;
import java.util.function.Predicate;

// Immutable envelope — safe to share across threads/subscribers
final class Message {
    final String id;
    final String topic;
    final Object payload;
    final long timestamp;
    Message(String topic, Object payload) {
        this.id = UUID.randomUUID().toString();
        this.topic = topic;
        this.payload = payload;
        this.timestamp = System.currentTimeMillis();
    }
}

// Consumer contract
interface Subscriber {
    void onMessage(Message m) throws Exception;
    default void onError(Message m, Throwable t) { /* log / metrics */ }
}

enum BackpressurePolicy { BLOCK, DROP_NEWEST, DROP_OLDEST }

// Binding of one subscriber to one topic: own queue + worker + filter
class Subscription {
    private final Subscriber subscriber;
    private final Predicate<Message> filter;           // content-based routing
    private final BlockingQueue<Message> queue;        // per-subscriber isolation
    private final BackpressurePolicy policy;
    private final int maxRetries;
    private volatile boolean running = true;

    Subscription(Subscriber s, Predicate<Message> filter, int capacity,
                 BackpressurePolicy policy, int maxRetries, ExecutorService pool) {
        this.subscriber = s;
        this.filter = filter != null ? filter : m -> true;
        this.queue = new ArrayBlockingQueue<>(capacity);
        this.policy = policy;
        this.maxRetries = maxRetries;
        pool.submit(this::runLoop);                    // one worker → FIFO ordering
    }

    void enqueue(Message m) {
        if (!filter.test(m)) return;                   // filtered out
        switch (policy) {
            case BLOCK:      try { queue.put(m); } catch (InterruptedException e)
                                 { Thread.currentThread().interrupt(); } break;
            case DROP_NEWEST: queue.offer(m); break;   // silently drop if full
            case DROP_OLDEST: while (!queue.offer(m)) queue.poll(); break;
        }
    }

    private void runLoop() {
        while (running) {
            try {
                Message m = queue.take();              // blocks; FIFO drain
                deliver(m);
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); return; }
        }
    }

    private void deliver(Message m) {
        for (int attempt = 0; attempt <= maxRetries; attempt++) {
            try { subscriber.onMessage(m); return; }   // at-least-once via retry
            catch (Exception ex) {
                if (attempt == maxRetries) { subscriber.onError(m, ex); /* -> DLQ */ }
                else { try { Thread.sleep((1L << attempt) * 50); }        // backoff
                       catch (InterruptedException ie) { Thread.currentThread().interrupt(); return; } }
            }
        }
    }

    void stop() { running = false; }
    Subscriber subscriber() { return subscriber; }
}

// A named channel that fans out to its subscriptions
class Topic {
    private final String name;
    private final Set<Subscription> subs = new CopyOnWriteArraySet<>(); // lock-free reads
    Topic(String name) { this.name = name; }

    Subscription subscribe(Subscriber s, Predicate<Message> filter,
                           int capacity, BackpressurePolicy p, int retries, ExecutorService pool) {
        Subscription sub = new Subscription(s, filter, capacity, p, retries, pool);
        subs.add(sub);
        return sub;
    }
    void unsubscribe(Subscription sub) { subs.remove(sub); sub.stop(); }

    void publish(Message m) { for (Subscription sub : subs) sub.enqueue(m); } // never blocks on slow sub
    boolean isEmpty() { return subs.isEmpty(); }
}

// Mediator + registry + lifecycle owner
class Broker {
    private final ConcurrentMap<String, Topic> topics = new ConcurrentHashMap<>();
    private final ExecutorService dispatchPool = Executors.newCachedThreadPool();

    private Topic topic(String name) { return topics.computeIfAbsent(name, Topic::new); }

    void publish(String topic, Object payload) { topic(topic).publish(new Message(topic, payload)); }

    Subscription subscribe(String topic, Subscriber s) {
        return subscribe(topic, s, null, 1000, BackpressurePolicy.BLOCK, 3);
    }
    Subscription subscribe(String topic, Subscriber s, Predicate<Message> filter,
                           int capacity, BackpressurePolicy policy, int retries) {
        return topic(topic).subscribe(s, filter, capacity, policy, retries, dispatchPool);
    }
    void unsubscribe(String topic, Subscription sub) {
        Topic t = topics.get(topic);
        if (t != null) t.unsubscribe(sub);
    }
    void shutdown() { dispatchPool.shutdownNow(); }
}
```

**Example input / output:**

```text
Setup:
  broker.subscribe("orders", subA)                          // logs orders
  broker.subscribe("orders", subB, filter=amount>1000, ...) // high-value only
  broker.subscribe("payments", subC)

broker.publish("orders",  {orderId:1, amount:500})
   -> subA.onMessage({amount:500})        // delivered (no filter)
   -> subB: filtered out (500 <= 1000)    // not delivered
   -> subC: different topic, not delivered

broker.publish("orders",  {orderId:2, amount:2599})
   -> subA.onMessage({amount:2599})
   -> subB.onMessage({amount:2599})       // passes filter

broker.publish("payments", {payId:9})
   -> subC.onMessage({payId:9})

Slow-subscriber case:
  subA is slow; its ArrayBlockingQueue(1000) fills.
  With policy=BLOCK, only the publisher publishing to "orders" waits on subA's queue;
  subB/subC and other topics keep flowing — isolation holds.
  With policy=DROP_OLDEST, publisher never blocks; subA loses oldest messages (best-effort).

Handler-throws case:
  subC.onMessage throws -> worker retries 3x with 50/100/200ms backoff
  -> still failing -> onError(m) called, message routed to DLQ; worker keeps running.
```

**Data:**

```

// Message envelope shape { "id": "8f2c...-uuid", "topic": "orders.created", "headers": { "region": "IN", "priority": "high" }, "key": "order-1042", "payload": { "orderId": 1042, "amount": 2599, "currency": "INR" }, "timestamp": 1751500000000 }

CODE
Copy
```yaml
# Per-subscription config knobs (extension points)
subscription:
  topic: orders.created
  filter: "headers.region == 'IN'"   # content-based routing
  queueCapacity: 1000
  backpressure: BLOCK                 # BLOCK | DROP_NEWEST | DROP_OLDEST
  delivery: AT_LEAST_ONCE            # AT_MOST_ONCE | AT_LEAST_ONCE
  maxRetries: 3
  deadLetterTopic: orders.created.DLQ
```

**Flow:**

1. 1
2. 1
3. 1
4. *
5. *
6. 1
7. Broker
8. -Map<String,Topic> topics
9. -ExecutorService pool
10. +publish(topic, payload)
11. +subscribe(topic, sub) : Subscription
12. +unsubscribe(topic, sub)
13. Topic
14. -String name
15. -Set<Subscription> subs
16. +publish(Message)
17. +subscribe(...)
18. +unsubscribe(sub)
19. Subscription
20. -Queue<Message> queue
21. -Predicate filter
22. -BackpressurePolicy policy
23. +enqueue(Message)
24. -runLoop()
25. -deliver(Message)
26. «interface»
27. Subscriber
28. +onMessage(Message)
29. +onError(Message, Throwable)
30. Message
31. +String id
32. +String topic
33. +Object payload
34. +long timestamp

