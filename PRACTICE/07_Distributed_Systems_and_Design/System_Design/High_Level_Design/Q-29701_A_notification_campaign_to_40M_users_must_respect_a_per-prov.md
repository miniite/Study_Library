---
type: "practice"
difficulty: "Deep"
---

# A notification campaign to 40M users must respect a per-provider send rate and complete within a 2-hour window — how do you design the pacing/throttle scheduler so you neither burst-limit the provider nor miss the window?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective pacing/throttle scheduler for sending notifications to 40 million users over a two-hour window while respecting per-provider send rates, we need to ensure both throughput and time efficiency. This involves understanding the constraints, breaking down the problem into manageable parts, and then implementing a strategy that balances rate limiting with completion within the given timeframe.

#### Triaging the Question

The core of this question is about designing a throttling mechanism that respects per-provider send rates (e.g., API rate limits) while ensuring all notifications are sent out within 2 hours. The key aspects being probed include:

Understanding the constraints: Identifying and quantifying the number of users, the time window, and the per-provider rate limit.
Strategic planning: Planning how to distribute the load across providers to avoid hitting the rate limits while ensuring timely completion.
Error handling and recovery: Considering potential failures and how to handle them.

#### First Principles Solution
Calculate the Total Number of Notifications: 40 million users.
Determine the Time Window: 2 hours (120 minutes).
Identify Per-Provider Rate Limits: Assume each provider has a rate limit, e.g., X requests per minute.

First, calculate the total number of notifications that can be sent in 2 hours given the per-provider rate limit: [ \text{Total Requests} = \text{Rate Limit (requests/min)} \times 120 ]

Next, distribute these requests across providers to avoid hitting the rate limits. If you have (N) providers, each provider should send approximately: [ \text{Requests per Provider} = \frac{\text{Total Requests}}{N} ]

To ensure timely completion within the two-hour window, we need to implement a scheduler that can handle these requests in a controlled manner. This involves queuing and rate-limiting at both the overall and provider levels.

Detailed Solution

Queue System:

Use an asynchronous queue system (e.g., Kafka, RabbitMQ) to manage the notifications.
Enqueue all 40 million notification tasks.

Rate-Limiting Mechanism:

Implement a rate-limiter at both the overall and provider levels.
At the overall level, control the total number of requests per minute.
At the provider level, ensure that no single provider sends more than its allowed limit.

Distribute Requests Evenly:

Use a load balancer to distribute tasks evenly across providers.
Implement a throttling mechanism at each provider's endpoint to enforce rate limits.

Monitoring and Logging:

Monitor the queue and rate-limiter to ensure compliance with constraints.
Log detailed information about sent notifications, errors, and delays.

Error Handling and Recovery:

Handle any failures by retrying failed tasks or notifying the user of issues.
Implement backoff strategies for rate limit exceeded scenarios.

#### Example
Workflow
Initialization: Start the queue with 40 million notifications.
Queue Processing:
Poll from the queue and send notifications to providers.
Enforce rate limits at both the queue and provider levels.
Completion Check: Continuously check if the two-hour window is approaching completion.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Ensure all constraints are respected (e.g., per-provider rates, total time).
Use a combination of queuing and rate-limiting to manage load effectively.
Implement robust error handling and recovery strategies.
Monitor the system closely during execution to catch and address any issues.

#### Common mistakes
include:

Ignoring provider-specific rate limits, leading to premature failures.
Failing to implement adequate queuing mechanisms, causing delays or loss of notifications.
Poorly designed retry policies leading to unnecessary load on the system.

#### Follow-ups
could involve discussing specific technologies used (e.g., Kafka for queuing), more detailed implementation strategies, and trade-offs between different design choices.

**Comparison:**

```
Provider	Rate Limit (req/min)
Provider1	50
Provider2	75
Provider3	65
```

**Code:**

```python
# Example code snippet for rate-limited queue processing

from ratelimiter import RateLimiter
import asyncio

async def send_notification(provider_id, notification):
    # Simulate sending a notification to a provider
    await asyncio.sleep(0.1)  # Simulated delay
    print(f"Notification sent to {provider_id}")

async def main():
    notifications = [f"Notification {i}" for i in range(40000000)]
    providers = ['Provider1', 'Provider2', 'Provider3']

    rate_limit_per_provider = 100  # Example rate limit
    total_time_window = 120  # Two hours

    tasks = []
    for notification in notifications:
        provider = providers[notification % len(providers)]
        task = send_notification(provider_id=provider, notification=notification)
        tasks.append(task)

    rate_limiter = RateLimiter(max_calls=len(notifications), period=total_time_window)

    # Process tasks with rate limiting
    for task in tasks:
        async with rate_limiter:
            await task

asyncio.run(main())
```

**Example input / output:**

```text
Input:
- Queue with 40 million notifications, 3 providers

Output:
- Notifications processed and sent according to provider rate limits within a two-hour window.
- Monitoring logs indicating successful sends and any errors or delays.
```

**Data:**

```json
{
  "notifications": [
    {"id": 1, "user_id": 1001, "provider": "Provider1", "message": "Notification for User 1"},
    {"id": 2, "user_id": 1002, "provider": "Provider2", "message": "Notification for User 2"}
    // ... up to 40 million notifications
  ],
  "providers": ["Provider1", "Provider2", "Provider3"]
}
```

**Flow:**

1. Yes
2. Exceeded
3. Not Exceeded
4. Time Out
5. Within Time
6. Initialize Queue
7. Queue Full?
8. Process Notifications
9. Check Rate Limits
10. Backoff and Retry
11. Schedule Next Batch
12. Monitor and Log
13. Completion Check
14. Complete with Errors
15. Successful Completion

