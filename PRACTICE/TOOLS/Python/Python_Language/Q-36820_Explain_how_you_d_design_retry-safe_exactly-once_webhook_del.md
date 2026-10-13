---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design retry-safe, exactly-once webhook delivery from your Python backend to customer endpoints, including signing, dedup, and dead-lettering.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a reliable webhook delivery system that ensures exactly-once delivery with retries, error handling, and deduplication in Python, we'll focus on key components such as signing to ensure authenticity, implementing an idempotent mechanism for deduplication, and setting up dead-letter queues (DLQs) for messages that fail multiple retries. The core idea is to break down the problem into manageable parts: message construction, signing, delivery with retries, deduplication, and error handling.

First, let's define what exactly-once means in this context. Exactly-once delivery ensures that each message is processed by the receiver exactly once, even if a network error or other issue causes multiple attempts at sending. This requires careful handling of state between retries to avoid processing duplicate messages.

#### Triage

The interviewer might be probing your understanding of:

Messaging protocols and patterns like Idempotent Retry.
Error handling mechanisms in Python.
Security practices such as message signing.
State management and idempotency.
Queue systems for dead-lettering.

#### Assumptions
The webhooks are HTTP POST requests with JSON payloads.
We use a modern queue system (e.g., AWS SQS, RabbitMQ) to manage retries and deduplication.
Each customer endpoint has a unique identifier (ID).
The webhook payload includes the necessary metadata for signing.

#### First-Principles Solution
Message Construction: Construct messages with all necessary data, including a timestamp, sequence number, and customer ID.
Signing: Use Python’s hmac or cryptography libraries to sign each message with a shared secret key known only by your backend and the endpoint.
Delivery with Retries: Use a queue system to manage retries. When a webhook fails due to network issues, the message should be put back into the queue for another attempt.
Deduplication: Implement idempotency at both ends (backend and receiver) using sequence numbers or timestamps.
Dead-Letter Queues: Set up DLQs in case messages fail multiple retries due to persistent issues.
Step-by-Step Implementation

Message Construction

PYTHON
def create_webhook_message(data, secret_key):
    message = json.dumps(data)  # Ensure data is JSON serializable
    return message

Signing

PYTHON
import hmac
from hashlib import sha256

def sign_message(message, secret_key):
    signature = hmac.new(secret_key.encode(), msg=message.encode(), digestmod=sha256).hexdigest()
    return signature

Delivery with Retries Using a queue system like AWS SQS or RabbitMQ:

PYTHON
import boto3

def send_to_queue(message, queue_url):
    sqs = boto3.client('sqs')
    response = sqs.send_message(
        QueueUrl=queue_url,
        MessageBody=message
    )
    return response['MessageId']

Deduplication

PYTHON
class Deduplicator:
    def __init__(self):
        self.seen = set()

    def is_unique(self, message_id):
        if message_id in self.seen:
            return False
        self.seen.add(message_id)
        return True

    def clean_up(self):
        self.seen.clear()

Dead-Letter Queues Set up DLQs in your queue system configuration to handle messages that fail multiple retries.

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure the signing mechanism is robust against tampering.
Implement proper error handling, logging, and monitoring for the entire pipeline.

#### Cons
ider load balancing and scaling considerations if dealing with high volumes of webhooks.
Follow best practices for managing secrets securely.

#### Common mistakes
might include failing to implement deduplication correctly or not setting up DLQs effectively. Always validate messages both on the sender and receiver sides to ensure exactly-once delivery.

**Comparison:**

```
Field	Value
Customer ID	12345
Timestamp	2023-10-10T12:00:00Z
Message	Order confirmation for customer 12345
```

**Code:**

```python
import json
from hashlib import sha256
import hmac
import boto3

def create_webhook_message(data, secret_key):
    message = json.dumps(data)
    return message

def sign_message(message, secret_key):
    signature = hmac.new(secret_key.encode(), msg=message.encode(), digestmod=sha256).hexdigest()
    return signature

def send_to_queue(message, queue_url):
    sqs = boto3.client('sqs')
    response = sqs.send_message(
        QueueUrl=queue_url,
        MessageBody=message
    )
    return response['MessageId']

class Deduplicator:
    def __init__(self):
        self.seen = set()

    def is_unique(self, message_id):
        if message_id in self.seen:
            return False
        self.seen.add(message_id)
        return True

    def clean_up(self):
        self.seen.clear()
```

**Example input / output:**

```text
# Example input and output

data = {
    "customer_id": "12345",
    "timestamp": "2023-10-10T12:00:00Z",
    "message": "Order confirmation for customer 12345"
}

secret_key = "your_secret_key_here"
message = create_webhook_message(data, secret_key)
signature = sign_message(message, secret_key)

# Send the message to a queue
queue_url = "http://example-queue-url.amazonaws.com/0123456789"
message_id = send_to_queue(f"{message}|{signature}", queue_url)

print(f"Message ID: {message_id}")
```

**Data:**

```json
{
  "data": {
    "customer_id": "12345",
    "timestamp": "2023-10-10T12:00:00Z",
    "message": "Order confirmation for customer 12345"
  }
}
```

**Flow:**

1. Yes
2. No
3. Create Webhook
4. Sign Message
5. Send to Queue
6. Queue Acknowledge
7. Retry?
8. Deduplicate and Process

