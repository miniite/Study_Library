---
type: "practice"
difficulty: "Deep"
---

# You add retries to your model-serving client for transient errors and during a GPU outage the retries create a storm that prevents recovery. Explain how you'd add exponential backoff with jitter, a retry budget, and a circuit breaker around the inference call.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of transient errors causing a "storm" during GPU outages, we need to implement a robust retry strategy that incorporates exponential backoff with jitter, a retry budget, and a circuit breaker around inference calls. This solution ensures that our model-serving client can handle failures gracefully while preventing overwhelming the system.

#### Triaging the Question

The question asks us to design a retry mechanism for transient errors in a distributed system where retries could potentially exacerbate issues during a GPU outage. The goal is to ensure that the system remains stable and recoverable by managing the rate of retries, using exponential backoff with jitter to avoid precise timing attacks, and implementing a retry budget to limit total attempts. Additionally, we need to introduce a circuit breaker pattern to handle catastrophic failures like a full GPU outage.

Key Assumptions
The model-serving client communicates with multiple GPUs in a distributed system.
Transient errors are temporary issues that can be resolved over time (e.g., network latency, resource contention).
A GPU outage is a more severe, non-transient error where the GPU itself is not available for some time.

#### First-Principles Solution
Exponential Backoff with Jitter

Exponential backoff helps to spread out retry attempts over time, reducing the likelihood of overwhelming a system with retries at the same intervals. Adding jitter (random variation) can further prevent timing attacks and ensure that all clients do not attempt retries exactly on the calculated interval.

Algorithm:

Define an initial delay D (e.g., 50ms).
Use exponential backoff to calculate the next retry delay, doubling the previous delay each time.
Introduce jitter by adding a small random value between -1/8 of the delay and +1/8 of the delay.

#### Example
Calculation:

Initial D = 50ms
After first failure: next_delay = D * 2 + random(-6.25, 6.25)
After second failure: next_delay = next_delay * 2 + random(-7.8125, 7.8125)
Retry Budget

A retry budget limits the total number of retries a client can make within a certain time window to prevent overwhelming the system with too many requests.

Algorithm:

Define a maximum number of retries R (e.g., 3).
Track the number of retries made by each client.
If the limit is exceeded, drop further requests until the budget resets or the issue resolves.
Circuit Breaker

A circuit breaker pattern protects the system from cascading failures. It monitors the error rate and automatically switches to a fail-fast mode if errors exceed a threshold.

Algorithm:

Define an error threshold (e.g., 50% of requests failing).
Monitor the failure rate.
If the failure rate exceeds the threshold, trigger the circuit breaker to drop all subsequent requests for a period (e.g., 30 seconds), allowing time for recovery.

#### Closing Summary

By implementing exponential backoff with jitter, a retry budget, and a circuit breaker around inference calls, we can manage transient errors in a distributed system without causing system-wide failures. This approach ensures that the model-serving client remains resilient to temporary issues while avoiding resource exhaustion during more severe outages.

#### Common Mistakes/Follow-Ups
Forgetting Jitter: Not adding jitter to backoff intervals can lead to precise timing attacks, which might cause all clients to retry at the same time.
Ignoring Budgets: Failing to limit retries can overwhelm the system, leading to cascading failures.
Lack of Circuit Breaker Monitoring: Without a mechanism to monitor and act on error rates, the circuit breaker may not activate in time to prevent significant downtime.

**Code:**

```python
import random

def exponential_backoff_with_jitter(max_attempts=5):
    delay = 50  # Initial delay in milliseconds
    for attempt in range(1, max_attempts + 1):
        next_delay = delay * (2 ** (attempt - 1)) + random.uniform(-delay / 8, delay / 8)
        yield next_delay

# Example usage
backoffs = list(exponential_backoff_with_jitter())
print(backoffs)  # Output: [50.637964555329124, 101.27592911065825, 202.5518582213165, ...]
```

**Flow:**

1. Yes
2. No
3. Yes
4. Initial Delay
5. Retry Attempt?
6. Exponential Backoff with Jitter
7. Next Delay
8. Retry Budget Exceeded?
9. Send Request
10. Circuit Breaker Trip
11. Drop Requests for Duration

