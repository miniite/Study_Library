---
type: "practice"
difficulty: "Deep"
---

# Design a rate-limited, cost-aware fan-out to send 10M push notifications in 15 minutes. How do you throttle to respect provider quotas and retry the failures without duplicate sends?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that sends 10 million push notifications in 15 minutes while respecting provider quotas, throttling, and avoiding duplicates, we need to consider several key aspects: rate limiting, parallel processing, failure handling, and retry mechanisms. The core idea is to distribute the load efficiently across multiple workers and ensure that each notification is processed exactly once.

First, let’s break down the question:

Rate Limiting: Respect provider quotas to avoid hitting API limits.
Fan-Out: Distribute notifications to multiple providers/services in parallel.
Throttling: Control the speed at which messages are sent to each provider.
Retry Mechanism: Ensure failed notifications are retried without duplicates.

Now, let’s walk through the solution step by step:

Define the Problem Scope:

We need to send 10 million push notifications in 15 minutes.
Provider quotas might limit the number of requests per second (RPS) or per minute.
Each notification should be processed exactly once.

Calculate Required RPS: [ \text{Total notifications} = 10,000,000 ] [ \text{Time duration} = 15 \text{ minutes} = 900 \text{ seconds} ] [ \text{Required RPS} = \frac{\text{Total notifications}}{\text{Time duration}} = \frac{10,000,000}{900} \approx 11,111 \text{ RPS} ]

Provider Quotas:

Assume a provider limits us to 5,000 RPS.
We need to distribute the load across multiple providers/services.

Throttling Strategy:

Throttle each service to avoid hitting the quota.
Use token bucket or leaky bucket algorithms for rate limiting.

#### Example
If we have 2 services, each should handle approximately ( \frac{11,111}{2} = 5,556 ) RPS.

Fan-Out Mechanism:

Implement a fan-out mechanism to distribute notifications across providers/services.
Use an event-driven architecture with message brokers like RabbitMQ or AWS SQS for efficient message passing.

Notification Processing Pipeline:

Store each notification in a database along with a unique identifier and state (e.g., created, sending, sent, failed).
Worker processes will pick up notifications from the queue, send them to the appropriate provider, and update the status in the database.

Retry Mechanism:

Implement exponential backoff for retries.
Use a deduplication mechanism (e.g., hash-based or sequence number) to ensure no duplicate sends.

#### Example
Store failed notifications temporarily and reprocess them after a delay, using a set of unique identifiers to avoid duplicates.

Failure Handling:

Log all failures and retries with detailed error messages.
Implement circuit breaker patterns for robustness against provider outages.

Monitoring and Metrics:

Monitor RPS, notification status, and failure rates.
Use metrics like success rate, failure rate, and average processing time to optimize the system.

Architecture Overview:

CODE
+-------------------+       +-------------------+
|                   |       |                   |
|  User Interface   |<----->|  Notification DB  |
|                   |       |                   |
+-------------------+       +-------------------+
                  ^
                  |
 +-----------------v-------------+
 |           Fan-Out              |
 |  (RabbitMQ, AWS SQS)          |
 +-----------------+-------------+
         ^      |
         |      v
    +----+-----+ +----+-----+
    |    Worker1       Worker2|
    +--------------------+----
                ^
                |
            +---+------+
            |  Provider1|
            +-----------+
                         ^
                         |
                     +---+------+ +---+------+
                     |   ProviderN| |   ProviderM|
                     +-------------+

**Comparison:**

```
Status	Description
Created	Notification created
Sending	Notification sent
Sent	Notification successfully sent
Failed	Notification failed to send
Retrying	Notification being retried
```

**Code:**

```python
# Example of a simple throttling function using Token Bucket Algorithm

import time
from collections import deque

class TokenBucket:
    def __init__(self, tokens_per_second, capacity):
        self.tokens = deque(maxlen=capacity)
        self.tokens_per_second = tokens_per_second
        self.last_time = time.time()

    def get_token(self):
        now = time.time()
        time_diff = now - self.last_time
        self.tokens.extend([1] * int(time_diff * self.tokens_per_second))
        self.last_time = now

        if len(self.tokens) < 1:
            return False
        else:
            self.tokens.popleft()
            return True

# Example usage
bucket = TokenBucket(tokens_per_second=5000, capacity=20)
for _ in range(10):
    if bucket.get_token():
        print("Token acquired")
    else:
        print("Rate limit exceeded")
```

**Example input / output:**

```text
Input:
- 10 million notifications to send in 15 minutes

Output:
- 10 million notifications successfully sent with no duplicates and within provider quotas.
- All failed notifications retried without duplicates.
- System metrics updated with success rate, failure rate, and average processing time.
```

**Data:**

```json
{
  "notifications": [
    {"id": 1, "device_id": "abc", "message": "Hello, User 1"},
    {"id": 2, "device_id": "def", "message": "Hello, User 2"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Notification Creation
6. Is Rate Limit Exceeded?
7. Log Failure
8. Send Notification
9. Is Retrying Needed?
10. Retry with Exponential Backoff
11. Update Status to Sent
12. Store Failed NotificationTemporarily
13. Reprocess Failed NotificationsAfter Delay

