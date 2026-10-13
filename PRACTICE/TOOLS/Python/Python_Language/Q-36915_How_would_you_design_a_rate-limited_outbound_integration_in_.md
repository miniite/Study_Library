---
type: "practice"
difficulty: "Deep"
---

# How would you design a rate-limited outbound integration in Python where the third party allows 10 req/s, coordinating the limit across all your worker instances?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a rate-limited outbound integration in Python that adheres to a third-party's allowance of 10 requests per second (req/s) and coordinates this limit across multiple worker instances, we need to ensure that no more than 10 requests are made to the external service from all our workers combined within any one-second window. This involves understanding the nature of rate limiting, ensuring fairness among concurrent requests, and handling potential race conditions.

The core idea is to implement a token bucket algorithm or use a simpler approach like batch scheduling with synchronization mechanisms such as semaphores. Let’s break down the solution from first principles:

Understand Rate Limiting: The third party allows 10 req/s, meaning no more than 10 requests can be sent in any one-second window.
Determine Coordination Mechanism: Since multiple instances might trigger requests simultaneously, we need a way to ensure that the total number of requests does not exceed the limit across all instances.

Now let's dive into the detailed solution:

Step-by-Step Solution
Token Bucket Algorithm: This is an efficient approach for implementing rate limiting.
Semaphore: Alternatively, use semaphores to control access to the external service.
Synchronization Mechanisms: Ensure that requests are batched and scheduled appropriately.
Token Bucket Implementation

We can implement a token bucket using Python’s queue module or a custom implementation. Here is a basic outline:

PYTHON
import time
from collections import deque

class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = deque()
        self.last_refill_time = time.time()

    def get_token(self) -> bool:
        current_time = time.time()
        if current_time > self.last_refill_time + 1.0 / self.refill_rate:
            self.tokens.append(1)
            self.last_refill_time = current_time
        while self.tokens and len(self.tokens) >= self.capacity:
            self.tokens.popleft()
        return bool(self.tokens)

def rate_limited_integration():
    bucket = TokenBucket(capacity=10, refill_rate=1.0)
    while True:
        if not bucket.get_token():
            time.sleep(0.05)  # Small delay to avoid busy-waiting
        # Make the request here
Semaphore Implementation

Using a semaphore can be simpler but less flexible for this scenario:

PYTHON
import threading

semaphore = threading.Semaphore(10)

def make_request():
    with semaphore:
        # Make the request here
        pass
Synchronization and Batch Scheduling

To ensure fairness among multiple instances, we need to coordinate across all workers. A common approach is to use a central service or a distributed coordination mechanism like a database or a message broker.

Centralized Coordination with Redis

Using Redis as a key-value store for tracking request counts:

PYTHON
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def rate_limited_integration():
    while True:
        if redis_client.incr('request_counter') <= 10:
            # Make the request here
            time.sleep(0.1)  # Simulate a delay to make multiple requests
        else:
            time.sleep(0.2)  # Sleep longer when limit is reached
Message Broker Approach with RabbitMQ

Using RabbitMQ for task queuing and rate limiting:

PYTHON
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

def make_request(ch, method, properties, body):
    # Make the request here
    channel.basic_ack(delivery_tag=method.delivery_tag)

channel.queue_declare(queue='rate_limited_queue')
channel.basic_consume(queue='rate_limited_queue', on_message_callback=make_request)
channel.start_consuming()

#### Closing Summary
and Common Mistakes
Common Mistake: Forgetting to synchronize requests properly can lead to exceeding the rate limit.
Mistake in Implementation: Not handling back-off or retries correctly, leading to service degradation.
Assumption of Fairness: Assuming that tokens are evenly distributed across instances without proper coordination.

By choosing an appropriate method (token bucket, semaphore, centralized coordination, etc.), we can ensure that the rate limit is respected and that all instances operate within the constraints provided by the third-party service.

**Comparison:**

| Time (s) | Tokens Available |
|----------|------------------|
| 0        | 3                |
| 0.1       | 4                |
| 1         | 10               |
| 1.1      | 9                 |

**Code:**

```python
import time
from collections import deque

class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = deque()
        self.last_refill_time = time.time()

    def get_token(self) -> bool:
        current_time = time.time()
        if current_time > self.last_refill_time + 1.0 / self.refill_rate:
            self.tokens.append(1)
            self.last_refill_time = current_time
        while self.tokens and len(self.tokens) >= self.capacity:
            self.tokens.popleft()
        return bool(self.tokens)

def rate_limited_integration():
    bucket = TokenBucket(capacity=10, refill_rate=1.0)
    while True:
        if not bucket.get_token():
            time.sleep(0.05)  # Small delay to avoid busy-waiting
        # Make the request here
```

**Example input / output:**

```text
# Example input and output for the rate_limited_integration function
integration_start_time = time.time()
requests_made = 0

for _ in range(10):
    if not TokenBucket.get_token():
        time.sleep(0.05)  # Small delay to avoid busy-waiting
    requests_made += 1

print(f"Total requests made: {requests_made}")
```

**Data:**

```json
{
    "capacity": 10,
    "refill_rate": 1.0,
    "tokens": [1, 2, 3],
    "last_refill_time": 1633072800.054321
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is token available?
7. Make request
8. Wait for 0.05s
9. Increment token count
10. Check if token limit reached
11. Increment request counter andwait until next second

