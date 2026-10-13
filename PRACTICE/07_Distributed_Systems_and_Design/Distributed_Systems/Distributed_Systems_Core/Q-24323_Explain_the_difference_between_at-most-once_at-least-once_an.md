---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between at-most-once at-least-once and exactly-once semantics with a concrete data-ingestion example of each and which you'd default to for a metrics-ingestion pipeline versus a billing events pipeline.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

At-most-once, at-least-once, and exactly-once are critical concepts in distributed systems when dealing with data ingestion and processing. At-most-once means that each event is processed no more than once; at-least-once guarantees that the event will be processed at least once but might be processed multiple times; exactly-once ensures that each event is processed precisely once, without duplication or omission.

In this answer, we'll explore these semantics through concrete examples in data ingestion pipelines. First, I’ll explain the concepts and then provide a specific example for both an metrics-ingestion pipeline (favoring at-least-once) and a billing events pipeline (favoring exactly-once).

#### Triage

The question is asking to explain three different processing guarantees—specifically in the context of data ingestion pipelines. We need to clarify that these semantics are critical when handling failure cases, such as network interruptions or node failures, which can lead to missing or duplicate messages.

#### Assumptions
and Context
Metrics-ingestion pipeline: Typically handles low-value data where occasional loss is acceptable but duplicate processing would cause issues.
Billing events pipeline: Handles high-stakes financial transactions where any failure could have significant consequences. Ensuring exactly-once semantics is crucial to avoid overcharging or undercharging.
At-most-once Semantics

At-most-once means that the system will not process a message more than once, even if it fails and retries. This can be achieved using mechanisms like atomic transactions but comes with the risk of missing messages in case of failures.

#### Example
In a metrics-ingestion pipeline where data is ingested from sensors measuring temperature or humidity levels, losing one reading might slightly skew the overall analysis. However, processing it again won't cause any further issues.

Yes
No
Message sent
Failure?
Lose message
Process message once
At-least-once Semantics

At-least-once guarantees that a message will be processed at least once, even if it is retried. However, this can lead to processing the same message more than once.

#### Example
In a metrics-ingestion pipeline, if there's an interruption in the data stream and the system retries sending a message, it might result in duplicate entries in the database.

Yes
Message sent
Failure?
Retry
Process message once
Exactly-once Semantics

Exactly-once semantics ensures that each message is processed exactly once, without duplication or omission. This is the most stringent guarantee but requires more complex and robust implementation.

#### Example
In a billing events pipeline where data represents financial transactions, ensuring exactly-once processing is critical. If a transaction for $100 is recorded twice due to network issues, it would lead to overcharging.

Yes
Message sent
Failure?
Retry with idempotentprocessing
Process message once
Metrics-Ingestion Pipeline (At-least-once)

For a metrics-ingestion pipeline, we prioritize at-least-once semantics because:

Occasional loss of low-value data is acceptable.
The cost of reprocessing to avoid duplicates is manageable.

#### Example
Imagine ingesting temperature readings from IoT devices. A small delay in processing one reading would not significantly impact the overall analysis of environmental conditions.

Billing Events Pipeline (Exactly-once)

For a billing events pipeline, we prioritize exactly-once semantics because:

Financial transactions must be processed precisely once to avoid overcharging or undercharging.
The stakes are high, and any failure can have severe financial implications.

#### Example
Consider processing payment transactions. If a $100 transaction is recorded twice due to a network interruption, it would result in double charging the customer, which could lead to significant legal and reputational issues for the company.

#### Closing Summary

In summary, at-least-once is suitable for scenarios where occasional data loss is acceptable but reprocessing is costly. Exactly-once is necessary when every event must be processed precisely once due to high stakes or financial implications. For metrics-ingestion pipelines, we might default to at-least-once because the cost of reprocessing can be mitigated by business logic. In contrast, billing events require exactly-once semantics to ensure accurate and consistent financial records.

#### Common Mistakes/Follow-Ups
Incorrect Choice of Semantics: Failing to understand the implications of each guarantee can lead to choosing an inappropriate processing strategy.
Implementing Exactly-Once Incorrectly: Ensuring correct exactly-once behavior requires careful design, often involving idempotent operations and coordination between nodes.

**Code:**

```
Example code for at-least-once processing (metrics-ingestion)

def process_temperature_data(reading): try: db.insert_reading(reading) except Exception as e: log.error(f"Failed to insert reading {reading}: {e}") # Implement retry logic here

CODE
Copy

```python
# Example code for exactly-once processing (billing events)
from idempotent_transaction import ensure_idempotent

@ensure_idempotent('transaction_id')
def process_payment(transaction):
    db.process_payment(transaction)
```

**Example input / output:**

```text
# Input: At-least-once Metrics Ingestion Example
input = {
    "sensor_id": "456",
    "value": 27.8,
    "timestamp": "2023-10-01T12:10:00Z"
}

# Output: Retry logic implemented for failure handling
retry_attempts = 3

for attempt in range(retry_attempts):
    try:
        db.insert_reading(input)
        print(f"Successfully inserted reading {input}")
        break
    except Exception as e:
        if attempt < retry_attempts - 1:
            print(f"Insertion failed, retrying... {e}")
        else:
            print(f"Failed to insert after {retry_attempts} attempts: {e}")

# Input: Exactly-once Billing Events Pipeline Example
input = {
    "transaction_id": "987654321",
    "amount": 200.0,
    "customer_id": "user456",
    "timestamp": "2023-10-01T12:10:00Z"
}

# Output: Idempotent processing ensures no duplicates
if ensure_idempotent('transaction_id'):
    db.process_payment(input)
    print(f"Successfully processed payment {input}")
else:
    print("Payment already processed or failed.")
```

**Data:**

```

No specific data schemas provided, but these are typical JSON formats:

JSON
Copy
# Metrics Ingestion Data Example
{
  "sensor_id": "123",
  "value": 25.4,
  "timestamp": "2023-10-01T12:00:00Z"
}

# Billing Events Pipeline Data Example
{
  "transaction_id": "123456789",
  "amount": 100.0,
  "customer_id": "user123",
  "timestamp": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Message sent
4. Failure?
5. Lose message
6. Process message once

