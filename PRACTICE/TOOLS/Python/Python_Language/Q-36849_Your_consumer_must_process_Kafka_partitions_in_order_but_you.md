---
type: "practice"
difficulty: "Deep"
---

# Your consumer must process Kafka partitions in order but you want parallelism. How would you structure Python workers to keep per-key ordering while scaling out?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of processing Kafka partitions in a specific order by key while still leveraging parallelism, we need to design a system that ensures messages with the same key are processed sequentially across workers. Here's how we can achieve this using Python:

TL;DR: We'll use a distributed task queue like Celery and Kafka to ensure per-key ordering while enabling parallel processing of partitions.

#### Triaging the Question

The core question is about implementing a system that processes messages from Kafka in such a way that all messages with the same key are processed sequentially, even when multiple workers are running. The goal is to balance between performance and maintaining order.

#### Assumptions
We have control over the Python backend.
We can integrate with external services like Celery for task orchestration.
Kafka partitions are already set up and understood by the consumer.

#### First Principles Solution
Kafka Consumers:
Use a single consumer group to ensure messages with the same key are delivered in order.
Task Queue Integration (Celery):
Distribute the tasks across multiple workers using Celery, which provides task chaining and prioritization capabilities.
Worker Design:
Each worker will consume messages from Kafka, process them sequentially by key, and then send the results to a result backend.
Step-by-Step Implementation

Kafka Consumer Setup:

PYTHON
from kafka import KafkaConsumer
from celery import Celery
import json

# Initialize Kafka consumer

#### cons
umer = KafkaConsumer(
    'your_topic',
    bootstrap_servers=['localhost:9092'],
    group_id='your_group_id'
)

# Initialize Celery app with a result backend (e.g., Redis)
app = Celery('tasks', broker='redis://localhost:6379/0', backend='redis://localhost:6379/0')

Processing Messages:

PYTHON
@app.task
def process_message(key, message):
    # Process the message based on key
    print(f"Processing {key} - {message}")

for msg in consumer:
    key = json.loads(msg.key).get('your_key_field')
    message = msg.value.decode()

    # Dispatch task to Celery with correct order
    process_message.delay(key, message)

Celery Worker:

PYTHON
from celery import Celery

app = Celery('worker', broker='redis://localhost:6379/0')

@app.task
def worker_task(key, message):
    # Consume messages and process them sequentially by key
    print(f"Worker processing {key} - {message}")

if __name__ == '__main__':
    app.start()

Result Backend:

Use Redis or another result backend to store the results of processed tasks.
PYTHON
from celery.result import AsyncResult

# Retrieve and process results
task_id = 'your_task_id'
result = AsyncResult(task_id)
print(result.get())

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we leverage Kafka for ordering messages by key and Celery to distribute tasks across multiple workers while maintaining the order. This approach ensures that all messages with the same key are processed sequentially even in a parallelized environment.

#### Common Mistakes

Not using a distributed task queue like Celery.
Failing to correctly configure Kafka consumers to respect partitioning.
Ignoring the need for a result backend to track task execution status.

#### Follow-Ups

#### Performance Considerations
How do we handle extremely high throughput? Would using multiple partitions or scaling out the worker pool help?
Fault Tolerance: What happens if a worker fails? Do we have retries and error handling in place?
Scalability: How can we ensure that our system scales horizontally without impacting ordering?

**Code:**

```

from kafka import KafkaConsumer import json

Initialize Kafka consumer

consumer = KafkaConsumer( 'your_topic', bootstrap_servers=['localhost:9092'], group_id='your_group_id' )

@app.task def process_message(key, message): # Process the message based on key print(f"Processing {key} - {message}")

for msg in consumer: key = json.loads(msg.key).get('your_key_field') message = msg.value.decode()

CODE
Copy
# Dispatch task to Celery with correct order
process_message.delay(key, message)
CODE
Copy

```python
from celery import Celery

app = Celery('worker', broker='redis://localhost:6379/0')

@ app.task
def worker_task(key, message):
    # Consume messages and process them sequentially by key
    print(f"Worker processing {key} - {message}")

if __name__ == '__main__':
    app.start()
PYTHON
Copy
from celery.result import AsyncResult

# Retrieve and process results
task_id = 'your_task_id'
result = AsyncResult(task_id)
print(result.get())
```

**Flow:**

1. yes
2. no
3. Consumer
4. Kafka Broker
5. Is key in cache?
6. Dispatch to worker usingCelery task
7. Store message temporarily
8. Worker processes messagessequentially by key

