---
type: "practice"
difficulty: "Deep"
---

# You must guarantee at-most-once processing of a webhook while the underlying insert is retried by the network layer; design the dedup key, the unique constraint, and the ON CONFLICT behavior that makes a retried delivery a no-op.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure at-most-once processing of a webhook in PostgreSQL while allowing retries from the network layer, we need to design a deduplication mechanism using a unique constraint and appropriate ON CONFLICT behavior. This approach ensures that each webhook is processed only once regardless of how many times it is delivered.

#### Triaging the Question

The core of the question revolves around ensuring idempotency in processing webhooks while allowing retries at the network layer. Specifically, we need to handle scenarios where a webhook might be delivered multiple times due to network issues but ensure that our database only processes each webhook once.

Key Assumptions and Requirements
Deduplication Key: This is an identifier that uniquely identifies a webhook.
Unique Constraint: To enforce the at-most-once processing, we use a unique constraint on the deduplication key.
ON CONFLICT Behavior: The ON CONFLICT behavior in PostgreSQL allows us to define what should happen when a unique constraint violation occurs.
Step-by-Step Solution
1. Define the Deduplication Key

We need a unique identifier for each webhook. A common approach is to use a combination of the webhook URL and a timestamp or sequence number. For instance, if we receive webhooks from different URLs, combining them with timestamps ensures uniqueness.

SQL
CREATE TABLE webhooks (
    id SERIAL PRIMARY KEY,
    url TEXT NOT NULL,
    received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    payload BYTEA NOT NULL,
    -- Other fields...
);
2. Add a Unique Constraint on the Deduplication Key

To enforce at-most-once processing, we add a unique constraint on the deduplication key.

SQL
ALTER TABLE webhooks ADD CONSTRAINT unique_webhook_key UNIQUE (url, received_at);

This ensures that even if the same webhook is delivered multiple times with slightly different received_at values, it will only be processed once.

3. Define ON CONFLICT Behavior

The ON CONFLICT DO NOTHING clause tells PostgreSQL to do nothing when a unique constraint violation occurs. This effectively means that any retry of the webhook delivery from the network layer will not result in an error or additional processing.

SQL
INSERT INTO webhooks (url, received_at, payload)
VALUES ('http://example.com/webhook', CURRENT_TIMESTAMP, 'webhook payload')
ON CONFLICT (url, received_at) DO NOTHING;

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing the above steps, we ensure that each webhook is processed at most once while allowing network-layer retries. The key points to remember are:

Use a unique combination of fields as the deduplication key.
Ensure the uniqueness constraint covers all possible duplicates.
Use ON CONFLICT DO NOTHING to make retry deliveries no-ops.

#### Common mistakes
include using non-unique keys or not handling conflicts properly, leading to duplicate processing. Follow-ups could involve discussing performance implications, indexing strategies for large-scale implementations, and considering distributed systems aspects if the system is highly concurrent.

