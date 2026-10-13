---
type: "practice"
difficulty: "Intermediate"
---

# How would you design an email notification system using a message queue?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Event-driven Order Processing](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Event-driven_Order_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an email notification system using a message queue, we'll leverage asynchronous processing to decouple our application components and ensure scalability, reliability, and performance. The core idea is to introduce a message broker that acts as an intermediary between the sending service (e.g., a user registration API) and the email service provider. This decoupling allows us to handle emails in batches, retry failed messages, and scale the system horizontally. Let's break down the design step by step.

**Code:**

```python
# Example code for producing a message to a queue
import json
from pika import BlockingConnection, ConnectionParameters

def send_email_notification(email_data):
    connection = BlockingConnection(ConnectionParameters('localhost'))
    channel = connection.channel()

    channel.queue_declare(queue='email_queue')
    channel.basic_publish(exchange='', routing_key='email_queue', body=json.dumps(email_data))
    print(f"Notification sent: {email_data}")
    connection.close()

# Example code for consuming messages from the queue
def consume_email_notifications():
    import pika

    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()

    channel.queue_declare(queue='email_queue', durable=True)
    def callback(ch, method, properties, body):
        email_data = json.loads(body)
        process_email_notification(email_data)
        ch.basic_ack(delivery_tag=method.delivery_tag)

    channel.basic_qos(prefetch_count=1)
    channel.basic_consume(queue='email_queue', on_message_callback=callback)
    print("Waiting for emails. To exit press CTRL+C")
    channel.start_consuming()

def process_email_notification(email_data):
    # Simulate sending email
    print(f"Processing: {email_data}")
```

