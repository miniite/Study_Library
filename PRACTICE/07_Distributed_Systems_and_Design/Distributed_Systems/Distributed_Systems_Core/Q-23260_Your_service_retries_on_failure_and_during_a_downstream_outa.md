---
type: "practice"
difficulty: "Deep"
---

# Your service retries on failure and during a downstream outage the retries create a retry storm that keeps the downstream from recovering. Explain the mechanism and the fixes: exponential backoff, jitter, retry budgets, circuit breakers.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, retries can sometimes lead to issues like "retry storms," where too many requests are sent to a failing service, exacerbating the problem instead of resolving it. This is particularly problematic when services depend on each other and a failure in one leads to an overwhelming number of retry attempts from its downstream services. To address this, several mechanisms—such as exponential backoff, jitter, retry budgets, and circuit breakers—are employed.

Firstly, exponential backoff ensures that retries are spaced out over time rather than being sent immediately after a failure. For example, if the initial delay is 1 second, subsequent delays might be 2 seconds, then 4 seconds, and so on. This helps prevent overwhelming the downstream service but can also cause delays in recovering from transient errors.

Jitter, or randomization, adds an unpredictable element to the retry intervals. By varying the backoff times slightly (e.g., adding a small random offset), you reduce the likelihood of multiple retries happening at exactly the same time, thus mitigating potential congestion issues.

A retry budget is another strategy where only a certain number of retries are allowed within a fixed time period. Once the budget is exhausted, no further retries are attempted for some duration. This helps manage resource usage and prevents overloading the system during high failure rates.

Finally, circuit breakers automatically stop sending requests to a failing service once it has been determined that the service is not available or unreliable. Circuit breakers can detect failures through health checks and trigger an open state where no further requests are sent until a period of stability is observed. This prevents a cascading failure across multiple services.

**Code:**

```python
# Example implementation of exponential backoff with jitter in Python

import time
from random import randint

def retry_with_backoff_and_jitter(max_retries, base_delay, max_delay):
    attempts = 0
    while attempts < max_retries:
        delay = min(base_delay * (2 ** attempts), max_delay)
        # Adding a small random jitter to the delay
        delay += randint(1, int(delay / 5))
        print(f"Retrying in {delay} seconds...")
        time.sleep(delay)
        attempts += 1

# Example usage
retry_with_backoff_and_jitter(max_retries=3, base_delay=1, max_delay=10)
```

**Example input / output:**

```text
Input: A service call that results in a failure.
Output: Retries are exponentially backoff with jitter added to the delay.

Initial Delay: 1 second
Retrying after 2 seconds (with jitter adjustment)
Retrying after 4 seconds (with jitter adjustment)
```

**Flow:**

1. No
2. Yes
3. Success
4. Fail
5. Service Call
6. Failure Detection
7. Retry Allowed?
8. Circuit Breaker Open
9. Exponential Backoff with Jitter
10. Attempt Retry
11. Check Success
12. Service Call Succeeded

