---
type: "practice"
difficulty: "Expert"
---

# Design an idempotency-and-exactly-once strategy that spans the entire request lifecycle across an API gateway, a queue, a worker, and a downstream payment processor. Where does each hop dedup, how do keys propagate, and where is exactly-once genuinely impossible so you must lean on idempotent effects instead?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotency-and-exactly-once strategy spanning from the API gateway to a downstream payment processor involves ensuring that each request is processed exactly once, while also handling retries and deduplication at various stages of the system. This solution spans multiple components including the API gateway, a queue system, a worker service, and a payment processor. Here’s how we can achieve this:

TL;DR
API Gateway: Initializes the request, generates a unique ID.
Queue System: Ensures deduplication and ordering of messages based on this ID.
Worker Service: Processes the message idempotently using the same unique ID.
Payment Processor: Applies exactly-once semantics by leveraging database transactions.

#### Triaging the Question

The core question is about designing a mechanism that ensures requests are processed exactly once across multiple system hops, while also managing retries and deduplication. The interviewer might be probing on:

Understanding of idempotency and exactly-once processing.
Knowledge of distributed systems and message queuing.
Ability to handle retries in asynchronous systems.

#### Assumptions
Idempotency: A request can be retried multiple times without changing the end result.
Exactly-once Semantics: Ensures that each request is processed exactly once, even in the presence of failures.
Retry Logic: In case of failures, retries should be managed to prevent duplicate processing.

#### First-Principles Solution
Step 1: API Gateway (Initial Request Handling)

The API gateway acts as a first point of contact and generates a unique request ID for each incoming request. This ensures that each request can be uniquely identified.

PYTHON
class ApiGateway:
    def __init__(self):
        self.request_id_generator = IdGenerator()

    def handle_request(self, request):
        request_id = self.request_id_generator.generate()
        # Forward request to queue with this unique ID
Step 2: Queue System (Deduplication and Ordering)

The queue system ensures that messages are processed in the order they were received and handles deduplication by using the generated request ID.

PYTHON
class QueueSystem:
    def __init__(self):
        self.message_ids = set()

    def enqueue(self, message_id):
        if message_id not in self.message_ids:
            self.message_ids.add(message_id)
            # Enqueue message for processing
Step 3: Worker Service (Processing Messages Idempotently)

The worker service processes the messages from the queue. Since it uses a unique ID, any duplicate requests will be handled idempotently.

PYTHON
class WorkerService:
    def __init__(self):
        self.processed_ids = set()

    def process_message(self, message_id, request_data):
        if message_id not in self.processed_ids:
            # Process the request data (e.g., update database)
            self.processed_ids.add(message_id)
Step 4: Payment Processor (Exactly-Once Semantics via Transactions)

The payment processor ensures that payments are processed exactly once by using transactions. If a transaction fails, it can be retried but will only succeed once.

PYTHON
class PaymentProcessor:
    def process_payment(self, payment_data):
        with self.db.transaction():
            # Process the payment (e.g., update account balance)

#### Common Mistakes
and Follow-Ups
Failing to Handle Retries: Not managing retries properly can lead to duplicate processing.
Insufficient Deduplication Logic: Ignoring deduplication in the queue or worker service can result in redundant requests being processed.
Ignoring Failures: Failing to account for transactional failures in the payment processor can result in partial processing of payments.

#### Closing Summary

By using a unique request ID at each hop and handling retries idempotently, we ensure that our system processes each request exactly once. The key is to manage deduplication and idempotent processing carefully across all components. Common pitfalls include insufficient retry management or failing to handle transactional failures properly.

**Comparison:**

| Component | Task                      |
|-----------|--------------------------|
| API Gateway | Generates request ID       |
| Queue System | Enqueues messages for processing |
| Worker Service | Processes idempotently   |
| Payment Processor | Ensures exactly-once via transactions |

**Code:**

```python
class ApiGateway:
    def __init__(self):
        self.request_id_generator = IdGenerator()

    def handle_request(self, request):
        request_id = self.request_id_generator.generate()
        # Forward request to queue with this unique ID

class QueueSystem:
    def __init__(self):
        self.message_ids = set()

    def enqueue(self, message_id):
        if message_id not in self.message_ids:
            self.message_ids.add(message_id)
            # Enqueue message for processing

class WorkerService:
    def __init__(self):
        self.processed_ids = set()

    def process_message(self, message_id, request_data):
        if message_id not in self.processed_ids:
            # Process the request data (e.g., update database)
            self.processed_ids.add(message_id)

class PaymentProcessor:
    def process_payment(self, payment_data):
        with self.db.transaction():
            # Process the payment (e.g., update account balance)
```

**Example input / output:**

```text
Input (API Gateway):
- Request Data

Output (Queue System):
- Unique Message ID (e.g., "1234567890abcdef")

Input (Worker Service):
- Enqueued Message with ID ("1234567890abcdef")

Output (Worker Service):
- Processed Message

Input (Payment Processor):
- Validated and Processed Payment Data

Output (Payment Processor):
- Updated Account Balance
```

**Data:**

```json
{
  "request_id_generator": {
    "type": "UUID"
  },
  "queue_system": {
    "message_ids": ["1234567890abcdef"]
  },
  "worker_service": {
    "processed_ids": ["1234567890abcdef"]
  }
}
```

**Flow:**

1. API Gateway
2. Generate Request ID
3. Queue System
4. Worker Service
5. Process Message
6. Payment Processor
7. Update Account Balance

