---
type: "practice"
difficulty: "Expert"
---

# Explain how you would move a hot write endpoint from synchronous DB write to write-behind through a queue while keeping the 201 Created contract and read-your-write semantics intact.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a hot write endpoint from synchronous database writes to a write-behind approach using a message queue, we need to ensure that the API continues to adhere to its 201 Created contract and maintains read-your-write semantics. The core idea is to decouple the request handling from immediate database writes by introducing an intermediary step involving a message queue.

#### Triage
Understanding the Problem

The question revolves around maintaining reliability, consistency, and performance in write-heavy operations while ensuring that clients receive timely responses. Specifically, we need to:

Preserve the 201 Created status code.
Ensure read-your-write semantics, meaning the client should be able to immediately or shortly after see their own writes reflected in reads.

#### First Principles Solution
Message Queue Setup: Introduce a message queue (e.g., RabbitMQ, Kafka) between the API and the database.
Asynchronous Processing: When an HTTP POST request arrives, the API will write the data to the queue rather than directly to the database.
Immediate Response Handling: The API should respond with 201 Created as soon as it has successfully written the message to the queue, indicating that the write operation was received by the system.
Queue Consumer: A background consumer will process messages from the queue and commit them to the database.
Detailed Solution
Step 1: Message Queue Setup

We'll use a message broker like RabbitMQ. The producer (API) sends data as JSON payloads to a dedicated write queue, while the consumer handles durable writes to the database.

Client
API Endpoint
Message Queue (e.g.,RabbitMQ)
Database Write Consumer
Step 2: Asynchronous Processing

When a client sends a POST request, the API:

Validates and Formats Request Data: Ensure data conforms to expected schema.
Serializes and Queues Message: Serialize request body into a JSON message format suitable for queuing.
Publishes to Queue: Publish this serialized message to a write queue.
PYTHON
# Example in Python using RabbitMQ
import pika

def send_to_queue(data):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='write_queue')

    # Serialize and publish data
    serialized_data = json.dumps(data)
    channel.basic_publish(exchange='', routing_key='write_queue', body=serialized_data)

    connection.close()

# API endpoint handling POST request
@app.route('/api/write', methods=['POST'])
def write():
    data = request.get_json()
    send_to_queue(data)
    return jsonify({"status": "201 Created"}), 201
Step 3: Immediate Response Handling

The API should immediately respond with 201 Created after the message is published to the queue. This indicates that the write operation was received and will be processed eventually.

PYTHON
# Example HTTP response
@app.route('/api/write', methods=['POST'])
def write():
    data = request.get_json()
    send_to_queue(data)

    # Respond immediately with 201 Created status
    return jsonify({"status": "201 Created"}), 201
Step 4: Queue Consumer

The queue consumer is responsible for reading messages from the write queue and writing them to the database. It should be designed to handle failures gracefully, ensuring eventual consistency.

PYTHON
# Example in Python using RabbitMQ
def consume_from_queue():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='write_queue')

    def callback(ch, method, properties, body):
        data = json.loads(body)
        # Handle data and commit to database
        print(" [x] Received %r" % data)

    channel.basic_consume(queue='write_queue', on_message_callback=callback, auto_ack=True)

    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':

#### cons
ume_from_queue()

#### Closing Summary
and Common Mistakes

By using a message queue, we maintain the 201 Created contract and ensure that write operations are decoupled from immediate database writes. The key is to handle messages asynchronously and commit them to the database in batches or at a later time.

#### Common Mistakes

Forgetting to handle retries and failures in the queue consumer.
Not acknowledging messages correctly, leading to duplicate processing.
Ignoring potential bottlenecks in message consumption and scaling issues.

**Code:**

```python
# API Endpoint handling POST request
@app.route('/api/write', methods=['POST'])
def write():
    data = request.get_json()
    send_to_queue(data)

    # Respond immediately with 201 Created status
    return jsonify({"status": "201 Created"}), 201

# Message queue producer function
import pika, json

def send_to_queue(data):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='write_queue')

    # Serialize and publish data
    serialized_data = json.dumps(data)
    channel.basic_publish(exchange='', routing_key='write_queue', body=serialized_data)

    connection.close()

# Queue consumer function
def consume_from_queue():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='write_queue')

    def callback(ch, method, properties, body):
        data = json.loads(body)
        # Handle data and commit to database
        print(" [x] Received %r" % data)

    channel.basic_consume(queue='write_queue', on_message_callback=callback, auto_ack=True)

    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':
    consume_from_queue()
```

**Example input / output:**

```text
# Example POST request input and output
request_body = {
    "id": 1,
    "user_id": 20,
    "data": {
        "field1": "value1",
        "field2": "value2"
    }
}

response = send_to_queue(request_body)
print(response)  # Output: {"status": "201 Created"}
```

**Data:**

```json
{
  "id": 1,
  "user_id": 20,
  "data": {
    "field1": "value1",
    "field2": "value2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client
4. API Endpoint
5. Message Queue (RabbitMQ)
6. Database Write Consumer
7. Database
8. Immediate Response?
9. 201 Created
10. Retry or Log Failure

