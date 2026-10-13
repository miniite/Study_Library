---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between Kafka and RabbitMQ, and when would you pick one over the other?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we'll compare Kafka and RabbitMQ, focusing on their core concepts, strengths, and use cases to determine when each might be preferred.

Kafka and RabbitMQ are both messaging systems but serve slightly different purposes in event-driven architectures. Kafka excels at handling large volumes of real-time data and is ideal for streaming applications where you need durable storage and fast processing. RabbitMQ, on the other hand, offers a more robust framework with flexible routing options and supports various communication patterns.

When deciding between them, consider factors like throughput requirements, data durability, and message complexity. Kafka’s distributed log structure makes it well-suited for scenarios needing high write performance and reliable replay of events, while RabbitMQ's broker-based architecture provides a rich set of features for complex workflows.

**Comparison:**

```
Feature	Kafka	RabbitMQ
Throughput	High for real-time data streams	Lower, but flexible with backpressure handling
Scalability	Easy to scale horizontally	More complex scaling
Durability	Guaranteed message delivery	Guaranteed per-queue and per-message
Complex Workflows	Limited	Supports many patterns (fanout, topic, headers)
Message Retention	Long-term storage via log compaction	Short-term with manual cleanup
```

**Code:**

```
Example of creating a topic in Kafka (Python with Confluent Kafka)

from confluent_kafka.admin import AdminClient

def create_topic(kafka_broker): admin_client = AdminClient({'bootstrap.servers': kafka_broker}) topics = ['example-topic']

CODE
Copy
fs = admin_client.create_topics(topics, config={'retention.ms': 604800000}) # 7 days
for topic, f in fs.items():
    try:
        f.result()
        print(f"Topic {topic} created")
    except Exception as e:
        print(f"Failed to create topic: {e}")
CODE
Copy

```python
# Example of setting up a connection in RabbitMQ (Python with Pika)
import pika

def connect_to_rabbitmq():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='example_queue')
    return connection, channel
```

**Example input / output:**

```
Example Kafka input/output

Input (producer): KafkaProducer.send('example-topic', key=b'key1', value=b'value1')

Output (consumer): while True: msg = consumer.poll(timeout_ms=1000) if msg is None or not msg.value(): continue print(msg.value().decode())

CODE
Copy

```plaintext
# Example RabbitMQ input/output
Input (publisher):
connection, channel = connect_to_rabbitmq()
channel.basic_publish(exchange='', routing_key='example_queue', body=b'message')

Output (consumer):
def on_message(channel, method_frame, header_frame, body):
    print(" [x] Received %r" % body)

channel.basic_consume(queue='example_queue', on_message_callback=on_message)
```

**Data:**

```json
{
  "kafka_topic_config": {
    "retention.ms": 604800000,
    "replication.factor": 3
  },
  "rabbitmq_queue_config": {
    "durable": true,
    "auto_delete": false
  }
}
```

**Flow:**

1. Kafka
2. RabbitMQ
3. Producer
4. Kafka or RabbitMQ?
5. Kafka Topic Creation andMessage Sending
6. RabbitMQ Queue Declarationand Basic Publish/Consume

