---
type: "practice"
difficulty: "Expert"
---

# Design a cross-system exactly-once flow: Kafka to a relational DB to an external payment gateway. Where do you place the idempotency boundary so the non-transactional external call never double-executes?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an exactly-once flow from Kafka, through a relational database (RDBMS), to an external payment gateway, we need to ensure that each message is processed precisely once and no operation is retried or lost. This involves placing idempotency boundaries correctly and handling retries in a way that avoids double-execution of the non-transactional external call.

At a high level, our approach will involve:

Ensuring Kafka's delivery semantics are used effectively.
Storing processed IDs in the RDBMS to prevent duplicate processing.
Handling idempotent calls to the payment gateway from within the RDBMS context.

The key challenge is ensuring that the external call remains idempotent, meaning it can be safely retried without causing a double execution.

**Code:**

```python
# Example Kafka consumer configuration
consumer = KafkaConsumer(bootstrap_servers='localhost:9092',
                        auto_offset_reset='earliest',
                        enable_auto_commit=True,
                        group_id='payment-processing-group')
```

**Example input / output:**

```text
No specific input-output examples are needed here, but we can outline a typical flow:
- Kafka message sent to a topic
- Message processed and its ID stored in the RDBMS
- External payment gateway call made only if the transaction is new
```

**Flow:**

1. Kafka Topic
2. Consumer Group
3. RDBMS (Transaction IDStorage)
4. External Payment Gateway
5. PaymentConfirmation/Rejection

