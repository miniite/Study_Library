---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a queue and a pub-sub system?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Contrast two messaging models — a queue (point-to-point / work-queue) and a pub-sub (publish-subscribe / fan-out topic). The interviewer wants you to explain the delivery semantics difference (competing consumers vs. broadcast), where each fits, and the nuance that modern brokers (Kafka, SQS+SNS, RabbitMQ) blur the line. Level is "intermediate experience," so I should go beyond the textbook definition into consumer groups, ordering, retries/DLQs, and concrete "when to use which" judgment.

What the interviewer is probing: Do you know that the core difference is who consumes a message — one worker out of many (queue) vs. every interested subscriber (pub-sub)? Can you reason about scaling consumers, decoupling, and when a single system does both?

Clarifying questions I'd state: Are we distributing work (do one thing once) or broadcasting an event (notify many independent systems)? Do we need ordering? At-least-once or exactly-once? These drive the choice.

Plan: Define each model → the one-line core difference → delivery semantics → scaling model → real systems and how they map → a combined pattern (fan-out to per-service queues) → ordering/retries/DLQ nuances → when to use which → summary.

The core difference in one line
Queue (point-to-point): a message is delivered to exactly one consumer among many competing consumers. The queue load-balances work. If you have 5 workers, each message is processed by one of them.
Pub-sub (publish-subscribe): a message published to a topic is delivered to every subscriber. The topic broadcasts an event. If 5 services subscribe, all 5 get their own copy.

Mental model: a queue is a to-do list workers pull from (each task done once); a topic is a radio broadcast (every listener hears it).

Delivery semantics

Queue / work-queue: Producers enqueue tasks; a pool of consumers pulls messages. The broker ensures each message goes to one consumer, typically at-least-once (message is redelivered if not acknowledged). This gives natural horizontal scaling of processing: add more workers and throughput rises, because they share the load. Order is usually per-queue FIFO but is broken once you have multiple concurrent consumers (unless you partition by key).

Pub-sub: Producers publish to a topic; the broker fans the message out to all current subscribers. Subscribers are decoupled — they don't know about each other or about the producer. Adding a new subscriber (e.g., a new analytics service) requires zero change to the producer. This is the backbone of event-driven architecture.

Scaling and coupling
Aspect	Queue	Pub-sub

#### Cons
umers per message	Exactly one	All subscribers
Add a consumer to scale	Splits load (faster)	Adds a full copy of the stream
Producer knows consumers?	No	No
Primary use	Distribute work	Broadcast events

In a queue, more consumers = more throughput. In pub-sub, more subscribers ≠ more throughput of any single subscriber's work — each subscriber still processes the whole stream. To scale a single subscriber's processing you combine the models (below).

How real systems map
RabbitMQ: has both. Direct/queue = point-to-point; a fanout exchange bound to multiple queues = pub-sub. The "topic" here is the exchange.
AWS SQS: a queue (point-to-point work distribution). AWS SNS: pub-sub (fan-out). The idiomatic AWS pattern is SNS → multiple SQS: publish once to SNS, fan out to a queue per consuming service, and each service scales its own workers on its own queue.
Apache Kafka: a log that unifies both. A topic is partitioned. A consumer group gives queue semantics — partitions are split across group members, so each message is processed once per group. Multiple different consumer groups on the same topic give pub-sub — each group independently reads the whole topic. So Kafka = pub-sub across groups, queue within a group. This is the key nuance an interviewer loves.
Google Pub/Sub / Azure Service Bus: Service Bus has "queues" (point-to-point) and "topics + subscriptions" (pub-sub) as first-class distinct concepts.
The combined pattern (this is what real systems build)

The powerful production pattern is pub-sub fan-out into per-consumer work queues:

Producer publishes an OrderPlaced event to a topic once.
The topic fans out to a queue owned by each interested service — email-svc-queue, inventory-svc-queue, analytics-svc-queue.
Each service scales its own worker pool off its own queue (queue semantics), fully isolated. A slow analytics consumer can't back-pressure the email service.

This gives you broadcast decoupling (pub-sub) plus independent scaling and isolation (queue) at the same time — the best of both.

Ordering, retries, and dead letters (intermediate-level details)
Ordering: Strict global order kills parallelism. Both models usually offer partition/key-based ordering — messages with the same key (e.g., same orderId) go to the same partition/consumer in order, while different keys process in parallel. SQS has FIFO queues (with MessageGroupId); Kafka orders within a partition.
Retries + DLQ: With at-least-once delivery, consumers must be idempotent (processing a duplicate must be safe). Failed messages that keep NACKing should be routed to a dead-letter queue after N attempts so one poison message doesn't block the pipeline.
Backpressure: In pub-sub fan-out, a slow subscriber must not stall the topic. Per-subscriber queues (or Kafka's per-group offsets) let each consumer lag independently.
Replay: A log-based system (Kafka) lets a new or recovered subscriber replay from an offset — you can add a consumer next month and reprocess history. A transient queue (classic SQS/RabbitMQ) deletes a message once acked; there's nothing to replay.
When to use which
Use a queue when the job is "do this task once" — image resizing, sending one email, processing one payment, background jobs. You want load-balanced workers and each task handled exactly once.
Use pub-sub when "something happened and multiple independent systems care" — UserSignedUp should trigger email + CRM + analytics + fraud check, each evolving independently.
Use both (fan-out to queues) when an event must reach several services and each service needs to scale/retry independently — the standard microservices event backbone.

#### Common mistakes
/ follow-up probes
Saying "queue = async, pub-sub = sync" — both are asynchronous; the difference is fan-out, not sync/async.
Forgetting idempotency under at-least-once delivery.
Using pub-sub broadcast when you actually needed a single consumer, causing duplicate side effects (two services both charging the card).

#### Follow-ups
"How does Kafka give you both?" (consumer groups vs. multiple groups). "How do you scale a pub-sub subscriber?" (per-subscriber queue/partitions). "At-least-once vs exactly-once?" (idempotency + dedup keys or transactional outbox).
Summary — what a strong answer emphasizes

The essential distinction is one consumer (queue, distribute work) vs. all consumers (pub-sub, broadcast events). Queues scale processing by adding competing workers; pub-sub scales reach by adding subscribers and decouples producers from an unknown set of consumers. Modern brokers unify them: Kafka is pub-sub across consumer groups and a work-queue within a group; the canonical microservices pattern is SNS/topic fan-out into one SQS/queue per service, combining broadcast decoupling with independent, isolated, scalable consumption — plus idempotency, DLQs, and key-based ordering to make it robust.

**Comparison:**

```
Dimension	Queue (point-to-point)	Pub-sub (topic)
Who consumes a message	Exactly one consumer	Every subscriber
Goal	Distribute work	Broadcast an event
Effect of adding consumers	Higher throughput (shared load)	Another full copy of the stream
Producer–consumer coupling	Decoupled	Decoupled (even to count of consumers)
Typical AWS service	SQS	SNS
RabbitMQ mapping	default/direct queue	fanout exchange → queues
Kafka mapping	one consumer group	multiple consumer groups
Classic use case	Background jobs, task processing	Event-driven microservices, notifications
```

**Code:**

```
Kafka: one topic, both semantics at once
Queue semantics WITHIN a group (each message processed once per group):
3 partitions, group "orders-workers" with 3 consumers -> each consumer
owns 1 partition; a message is handled by exactly ONE group member.

kafka-console-consumer --topic orders --group orders-workers ...

Pub-sub semantics ACROSS groups (each group gets the WHOLE stream):

kafka-console-consumer --topic orders --group email-svc ... # full copy kafka-console-consumer --topic orders --group analytics-svc ... # full copy

CODE
Copy

```yaml
# AWS canonical fan-out: SNS (pub-sub) -> per-service SQS (queue)
Topic: order-events                 # SNS: publish OrderPlaced once
Subscriptions:
  - protocol: sqs
    endpoint: email-svc-queue       # SQS: email workers scale here
  - protocol: sqs
    endpoint: inventory-svc-queue   # SQS: inventory workers scale here
  - protocol: sqs
    endpoint: analytics-svc-queue   # SQS: analytics workers scale here
```

**Example input / output:**

```

Queue (point-to-point) — one message, 3 workers:

CODE
Copy
Producer -> [ m1 m2 m3 m4 ]  (queue)
Worker A takes m1, m4
Worker B takes m2
Worker C takes m3
=> each message processed EXACTLY ONCE (load balanced)

Pub-sub (topic) — one message, 3 subscribers:

CODE
Copy
Producer -> "OrderPlaced" (topic)
   -> email-svc      gets a copy  (sends email)
   -> inventory-svc  gets a copy  (decrements stock)
   -> analytics-svc  gets a copy  (records metric)
=> EVERY subscriber processes the SAME event
```

**Data:**

```json
// Event published once to the topic
{
  "eventType": "OrderPlaced",
  "orderId": "ORD-10231",
  "userId": "U-88",
  "amount": 1499.00,
  "occurredAt": "2026-07-07T10:15:00Z"
}
```

**Flow:**

1. Pub-Sub fan-out into per-service queues
2. Producer
3. {Topic: OrderPlaced}
4. (email queue)
5. email-svc
6. (inventory queue)
7. inventory-svc
8. (analytics queue)
9. analytics-svc
10. Queue: point-to-point
11. Producer
12. (Work Queue)
13. Worker A
14. Worker B
15. Worker C

