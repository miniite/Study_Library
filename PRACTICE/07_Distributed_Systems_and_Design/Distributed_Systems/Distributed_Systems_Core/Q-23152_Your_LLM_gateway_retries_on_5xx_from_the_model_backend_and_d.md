---
type: "practice"
difficulty: "Intermediate"
---

# Your LLM gateway retries on 5xx from the model backend, and during a GPU outage those retries form a storm that keeps the backend from recovering. Explain the mechanism and how exponential backoff with jitter, a retry budget, and a circuit breaker break it.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The issue described is a classic scenario in distributed systems where retries can lead to storming during backend outages. The solution involves implementing exponential backoff with jitter, a retry budget, and a circuit breaker to mitigate this problem.

Mechanism Explanation

When a 5xx error occurs from the LLM model backend, your gateway immediately retries the request. If the same issue persists due to an ongoing GPU outage, these retries can overwhelm the backend, preventing it from recovering. This is known as a "retry storm."

Key Concepts and Assumptions
Exponential Backoff with Jitter: A technique that increases the delay between retries exponentially but adds some randomness (jitter) to avoid synchronization.
Retry Budget: Limits the number of retries or the total number of requests that can be retried in a given time period.
Circuit Breaker Pattern: Automatically isolates failing services, stopping further calls and triggering fallback actions.

#### First Principles Solution

Exponential Backoff with Jitter:

Basic Mechanism: Each failed request triggers an exponential backoff, increasing the delay between retries. For example, you might start with a small delay (e.g., 50ms) and double it on each retry (50ms -> 100ms -> 200ms -> ...).
Jitter: Introduce random delays within the backoff window to avoid thundering herds. For instance, jitter could add a small random value between 0-33% of the delay.

#### Example
in Code:
PYTHON
import time

def exponential_backoff_with_jitter(retries=5):
    base_delay = 50  # Base delay in milliseconds
    for i in range(1, retries + 1):
        sleep_time = base_delay * (2 ** (i - 1)) * (random.uniform(0.7, 1))
        time.sleep(sleep_time)

Retry Budget:

Basic Mechanism: Limit the total number of retries or requests that can be retried in a specific period.

#### Example
Allow only 3 retries per minute for each backend instance.

#### Implementation
Maintain a counter for each request and reset it periodically.

Circuit Breaker:

Basic Mechanism: Once the failure rate exceeds a certain threshold, the circuit breaker trips, stopping further requests to the backend and triggering fallback mechanisms.
Threshold Calculation: If more than 80% of recent requests fail over a rolling window (e.g., 1 minute), trip the circuit breaker.
Fallback Mechanism: Provide a safe alternative for the user while the backend recovers, such as returning cached data or default values.

#### Closing Summary
and Common Mistakes
Summary: By implementing exponential backoff with jitter, managing retry budgets, and using circuit breakers, you can effectively mitigate retry storms during backend outages.

#### Common Mistakes
Failing to implement jitter in the backoff mechanism, leading to synchronized retries and thundering herds.
Setting an overly generous retry budget, which could delay real recovery of the backend if it is truly down.
Not tripping the circuit breaker early enough or allowing too many failed requests before taking action.
Practical Considerations
Monitoring: Continuously monitor failure rates to quickly identify and act on issues.
Feedback Loop: Implement a feedback loop where the system learns from past failures and dynamically adjusts backoff parameters.

