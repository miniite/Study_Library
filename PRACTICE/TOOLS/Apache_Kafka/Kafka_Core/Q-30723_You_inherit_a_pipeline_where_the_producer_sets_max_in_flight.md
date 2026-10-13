---
type: "practice"
difficulty: "Deep"
---

# You inherit a pipeline where the producer sets max.in.flight=5 with retries and no idempotence. Explain the concrete failure you'd expect in production and the one-line fix.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent and Transactional Producers](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this Kafka configuration issue, let's first understand the key aspects of the scenario: a producer configured with max.in.flight=5 without idempotence and retries enabled. In production, we would likely encounter failures due to the non-idempotent nature of these operations. Specifically, if multiple in-flight messages result in partial delivery or failure, it can lead to inconsistent states across consumers.

#### Triaging the Question

The question is probing our understanding of Kafka's message handling mechanics and potential pitfalls related to transactional semantics. The key assumptions are:

max.in.flight=5 implies that up to 5 messages can be sent in-flight simultaneously.
Retries are enabled, which means failed operations will attempt to retry.
Idempotence is not enforced, meaning the same operation can produce different results.

#### First-Principles Solution

When a producer sends max.in.flight=5 messages and these fail due to partial delivery or other reasons, we could encounter the following failure scenario:

Partial Delivery: Suppose 3 out of 5 in-flight messages are successfully delivered while the remaining 2 fail.
Retry Mechanism: Retries will attempt to retransmit the failed messages, but this could lead to duplicate deliveries if not managed correctly.

#### Closing Summary
and Common Mistakes/Follow-Ups

The common mistake here is failing to ensure idempotence in operations that are retried. Without idempotence guarantees, you risk inconsistent states or data corruption when partial failures and retries occur. A one-line fix would be to enforce idempotence in the producer's logic.

#### Example
One-Line Fix:
PYTHON
def send_message(message):
    # Ensure message is idempotent here.
    ...
Closing Thoughts

Remember, ensuring idempotence is crucial for maintaining data integrity and consistency in distributed systems. In Kafka, this can often be achieved through proper message design or by leveraging transactional semantics if supported.

