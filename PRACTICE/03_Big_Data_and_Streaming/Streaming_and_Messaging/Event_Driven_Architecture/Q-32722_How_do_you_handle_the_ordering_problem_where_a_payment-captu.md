---
type: "practice"
difficulty: "Deep"
---

# How do you handle the ordering problem where a payment-captured event can be processed before the order-created event that logically precedes it, across two topics?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-driven Order Processing](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Event-driven_Order_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the ordering problem in an event-driven architecture (EDA) where a payment-captured event might be processed before its corresponding order-created event, we need to ensure that messages are processed in the correct order despite potential delays or out-of-order delivery. This is a common issue when dealing with distributed systems and can lead to inconsistencies if not handled properly.

#### Triage
Restating the Question

The question asks how to handle ordering problems across two topics (order-created and payment-captured) where messages from one topic might arrive before their corresponding messages in another topic. The key aspect being probed is our understanding of EDA principles, message sequencing strategies, and potential solutions that can ensure logical order processing.

#### Assumptions
Event Topics: We have two topics: order-created for new orders and payment-captured for payment confirmations.
Message Correlation: Each event has a unique identifier or correlation ID that links it to its corresponding event in the other topic.
Latency Variability: There can be variability in message delivery times due to network delays, processing time differences, etc.

#### First-Principles Solution

To solve this ordering problem, we need to implement mechanisms that ensure messages are processed in a way that respects their logical order. This involves:

Correlation ID Tracking: Each event message includes a unique correlation ID.
Message Sequence Management: Maintaining a sequence of events based on these IDs.
Reordering Mechanisms: Implementing strategies to reorder and deduplicate messages if they arrive out of order.
Step-by-Step Solution

Correlation ID Assignment:

Each event message should include a unique correlation ID that is consistent across both topics for the same logical entity (e.g., an order).

Message Sequence Tracking:

A centralized service or distributed storage can keep track of the sequence of events based on these IDs.
For example, using a database table to log when each event type (order-created, payment-captured) has been processed.

Message Reordering Mechanism:

When an out-of-order message arrives, it is stored in a buffer or a temporary queue.
Periodically, the system checks for missing events and reorders them based on their correlation IDs.

Handling Duplicates:

Use deduplication mechanisms to ensure that duplicate messages do not cause processing issues.

#### Implementation
Example

Let's consider an implementation where we use Apache Kafka for messaging and a PostgreSQL database for sequence tracking.

Sample Schema
SQL
CREATE TABLE order_events (
    id SERIAL PRIMARY KEY,
    correlation_id VARCHAR(100) NOT NULL UNIQUE,
    event_type VARCHAR(50) NOT NULL, -- 'order-created', 'payment-captured'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Kafka Topic Setup
Two topics: order-created and payment-captured.
Each message includes a correlation ID.
Sequence Tracking Service (Python Example)
PYTHON
import psycopg2

def track_event(correlation_id, event_type):
    conn = psycopg2.connect(database="your_db", user="user", password="password", host="localhost", port="5432")
    cursor = conn.cursor()

    try:
        # Check if the event is already tracked
        cursor.execute("SELECT id FROM order_events WHERE correlation_id=%s AND event_type=%s", (correlation_id, event_type))
        result = cursor.fetchone()

        if not result:
            # Insert new event into the database
            cursor.execute("INSERT INTO order_events (correlation_id, event_type) VALUES (%s, %s)", (correlation_id, event_type))
            conn.commit()
    finally:
        cursor.close()
        conn.close()

def handle_out_of_order_message(correlation_id):
    # Reorder and process the message
    pass
Kafka Consumer Logic
PYTHON
from kafka import KafkaConsumer

# Create a consumer for each topic

#### cons
umer1 = KafkaConsumer('order-created', bootstrap_servers=['localhost:9092'])

#### cons
umer2 = KafkaConsumer('payment-captured', bootstrap_servers=['localhost:9092'])

for msg in consumer1:
    correlation_id = msg.value['correlation_id']
    event_type = 'order-created'

    track_event(correlation_id, event_type)
    handle_out_of_order_message(correlation_id)

for msg in consumer2:
    correlation_id = msg.value['correlation_id']
    event_type = 'payment-captured'

    track_event(correlation_id, event_type)
    handle_out_of_order_message(correlation_id)

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure correct ordering of events in an EDA setup:

Use correlation IDs to link related events.
Implement a sequence tracking mechanism using a database or other storage system.
Employ reordering mechanisms to process out-of-order messages.

#### Common mistakes
include:

Ignoring the need for deduplication, which can lead to processing the same message multiple times.
Failing to implement a robust sequence management system that can handle delays and outages.

#### Follow-up
questions might explore specific technologies used (e.g., Kafka, RabbitMQ), how these solutions scale with increasing traffic, or how they interact with other parts of the system like databases and storage systems.

**Comparison:**

```
id	correlation_id	event_type	created_at
1	abcd1234	order-created	2023-10-05 14:00:00
2	abcd1234	payment-captured	2023-10-06 10:00:00
```

**Code:**

```python
import psycopg2

def track_event(correlation_id, event_type):
    conn = psycopg2.connect(database="your_db", user="user", password="password", host="localhost", port="5432")
    cursor = conn.cursor()

    try:
        # Check if the event is already tracked
        cursor.execute("SELECT id FROM order_events WHERE correlation_id=%s AND event_type=%s", (correlation_id, event_type))
        result = cursor.fetchone()

        if not result:
            # Insert new event into the database
            cursor.execute("INSERT INTO order_events (correlation_id, event_type) VALUES (%s, %s)", (correlation_id, event_type))
            conn.commit()
    finally:
        cursor.close()
        conn.close()

def handle_out_of_order_message(correlation_id):
    # Reorder and process the message
    pass

# Kafka Consumer Logic Example
from kafka import KafkaConsumer

consumer1 = KafkaConsumer('order-created', bootstrap_servers=['localhost:9092'])
consumer2 = KafkaConsumer('payment-captured', bootstrap_servers=['localhost:9092'])

for msg in consumer1:
    correlation_id = msg.value['correlation_id']
    event_type = 'order-created'

    track_event(correlation_id, event_type)
    handle_out_of_order_message(correlation_id)

for msg in consumer2:
    correlation_id = msg.value['correlation_id']
    event_type = 'payment-captured'

    track_event(correlation_id, event_type)
    handle_out_of_order_message(correlation_id)
```

**Example input / output:**

```

Sample Input:

Order Created:

JSON
Copy
{
  "correlation_id": "abcd1234",
  "event_type": "order-created"
}

Payment Captured:

JSON
Copy
{
  "correlation_id": "abcd1234",
  "event_type": "payment-captured"
}

Output:

Track event for order-created and store in database.
Track event for payment-captured and store in database.
Handle out-of-order message if necessary.
SQL
Copy
INSERT INTO order_events (correlation_id, event_type) VALUES ('abcd1234', 'order-created');
INSERT INTO order_events (correlation_id, event_type) VALUES ('abcd1234', 'payment-captured');
```

**Data:**

```sql
CREATE TABLE order_events (
    id SERIAL PRIMARY KEY,
    correlation_id VARCHAR(100) NOT NULL UNIQUE,
    event_type VARCHAR(50) NOT NULL, -- 'order-created', 'payment-captured'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. Order Created
2. Track Event
3. Payment Captured
4. Track Event
5. Deduplicate if necessary
6. Deduplicate if necessary
7. Order Processing Logic
8. Order Processing Logic

