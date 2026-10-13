---
type: "practice"
difficulty: "Deep"
---

# How would you implement a token-bucket rate limiter in Python backed by Redis so it works correctly across multiple app instances, and why is a naive in-memory limiter wrong?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a token-bucket rate limiter in Python using Redis, we need to ensure that the system can handle distributed state, meaning it should work correctly across multiple app instances. A naive in-memory approach would not scale and could lead to inconsistencies. Here’s how you can build this robust solution.

Firstly, let's understand what a token-bucket rate limiter does: It allows a certain number of tokens (requests) per second while replenishing them over time. When a request comes in, the system checks if there are enough tokens available; if so, it deducts one token and returns success. Otherwise, it denies the request.

#### Triaging the Question

The key points we need to address:

Distributed State Management: How can multiple app instances share a consistent state?
Token Bucket Algorithm: Implementing the algorithm correctly.
Redis Integration: Using Redis as a backend for token storage and updating.

#### First-Principles Solution

We'll use Redis to store the number of tokens, their last update timestamp, and the refill rate. Each app instance will interact with this shared state using Redis commands to check and update tokens.

Step 1: Setup Redis Client

First, we need a Redis client that is thread-safe and can handle multiple connections across instances. redis-py is a popular choice for Python.

PYTHON
import redis
from threading import Lock

client = redis.Redis(host='localhost', port=6379, db=0)
lock = Lock()
Step 2: Define the Token Bucket Class

We'll create a class to manage tokens and interactions with Redis.

PYTHON
class TokenBucket:
    def __init__(self, token_bucket_key, refill_rate, capacity):
        self.key = token_bucket_key
        self.refill_rate = refill_rate
        self.capacity = capacity

    def get_tokens(self, app_instance_id):
        # Lock to ensure thread safety
        with lock:
            now = int(time.time())
            with client.pipeline() as pipe:
                pipe.watch(f'{self.key}:{app_instance_id}')
                token_count, last_update_time = pipe.get(f'{self.key}:{app_instance_id}'), pipe.get(f'{self.key}:last-update')
                if pipe.execute()[0] is None:  # Key doesn't exist, create it
                    pipe.multi()
                    pipe.set(f'{self.key}:{app_instance_id}', self.capacity)
                    pipe.set(f'{self.key}:last-update', now)
                    pipe.execute()

                token_count = int(token_count.decode())
                last_update_time = int(last_update_time.decode())

                # Refill tokens based on the refill rate
                if (now - last_update_time) >= 1:
                    refill_amount = self.refill_rate * (now - last_update_time)
                    available_tokens = min(self.capacity, token_count + refill_amount)
                    pipe.multi()
                    pipe.set(f'{self.key}:{app_instance_id}', available_tokens)
                    pipe.set(f'{self.key}:last-update', now)

                pipe.execute()

            return token_count
Step 3: Check and Deduct Tokens

When a request comes in, we need to check if there are enough tokens. If so, deduct one token.

PYTHON
def handle_request(app_instance_id):
    bucket = TokenBucket('my-token-bucket', refill_rate=10, capacity=20)
    available_tokens = bucket.get_tokens(app_instance_id)

    with lock:
        with client.pipeline() as pipe:
            pipe.watch(f'my-token-bucket:{app_instance_id}')
            if int(pipe.get(f'my-token-bucket:{app_instance_id}')) > 0:
                pipe.multi()
                pipe.decr(f'my-token-bucket:{app_instance_id}', count=1)
                pipe.execute()

    return "Success" if available_tokens else "Rate Limited"

#### Closing Summary

This implementation ensures that the token bucket rate limiter works correctly across multiple app instances by leveraging Redis for shared state management. The use of a lock and transactions helps maintain thread safety and ensure atomic updates.

#### Common Mistakes/Follow-Ups
Inconsistent State: Not using a distributed store like Redis can lead to inconsistencies.
Thread Safety: Ensure that the code handles concurrent requests safely, as shown with locks and transactions.
Rate Calculation: Precision in calculating refill rates is crucial; otherwise, the rate limiter may be inaccurate.

