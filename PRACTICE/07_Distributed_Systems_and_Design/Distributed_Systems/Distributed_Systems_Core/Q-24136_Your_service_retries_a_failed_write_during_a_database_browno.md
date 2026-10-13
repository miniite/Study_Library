---
type: "practice"
difficulty: "Deep"
---

# Your service retries a failed write during a database brownout and the retry storm keeps the DB from recovering. Explain the mechanism and the fixes: exponential backoff with jitter, a retry budget, and a circuit breaker around the query.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a retry storm causing a database to fail to recover during a brownout condition, we need to implement three key mechanisms: exponential backoff with jitter, a retry budget, and a circuit breaker around the query.

Firstly, let's understand the core problem. During a brownout, the database might be temporarily unavailable or operating at reduced capacity. Retrying failed writes can exacerbate the issue if too many retries occur simultaneously (a "retry storm"), overwhelming the system further and preventing recovery.

Mechanism Explained
1. Exponential Backoff with Jitter

Exponential backoff is a technique that gradually increases the time between retry attempts to prevent overwhelming the system. Jitter can be added to this mechanism to randomize the delay, which helps in spreading out retries across multiple instances or users.

#### Example

Initial delay: 1 second
Delay after each failure: 2^n * initial_delay + random_jitter, where n is the retry count

This means:

Retry 1: ~1s (with some jitter)
Retry 2: ~3-5s (randomized between 3 and 5 seconds due to jitter)
Retry 3: ~7-15s
...
2. Retry Budget

A retry budget involves setting a limit on the number of retries allowed before taking more drastic actions, such as failing the request or circuit breaking.

#### Example

Maximum retries per second: 100
If too many retries occur in one second, the budget is exhausted and no further retries are attempted until it resets after a certain period (e.g., 5 minutes).
3. Circuit Breaker

A circuit breaker acts as a safety mechanism that prevents overwhelming the database by failing fast if a threshold of failures occurs within a given time frame.

#### Example

Failure threshold: 100 failed writes in one minute
Once the threshold is reached, the circuit breaker trips and sends all subsequent requests to a fallback or failure response (e.g., returning an error message) until it resets after some period (e.g., 5 minutes).

#### Implementation
Steps
Step 1: Exponential Backoff with Jitter

Implement exponential backoff logic in your retry handler. Use a random jitter function to add variability to the delay times.

PYTHON
def calculate_backoff(retry_count, initial_delay=1):
    max_jitter = initial_delay * 0.5
    return (2 ** retry_count) * initial_delay + random.uniform(0, max_jitter)
Step 2: Retry Budget

Track the number of retries per second and enforce a budget.

PYTHON
retry_budget_per_second = 100
current_retries_this_second = 0

def handle_retry(retry_count):
    global current_retries_this_second
    if retry_count > retry_budget_per_second:
        # Handle budget exceeded, e.g., circuit break or fail fast
        return "Circuit breaker tripped"

    current_retries_this_second += 1

    # Calculate backoff with jitter and perform the retry
    delay = calculate_backoff(retry_count)
    time.sleep(delay)
    # Perform database operation here...
    if operation_succeeded:
        reset_retry_budget()
    else:
        increment_retry_count()
Step 3: Circuit Breaker

Implement a circuit breaker to monitor failures and trip when the threshold is exceeded.

PYTHON
class CircuitBreaker:
    def __init__(self, failure_threshold=100, reset_timeout=5 * 60):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.last_failure_time = None
        self.failures_count = 0

    def handle_request(self):
        if self.should_trip():
            return "Circuit breaker tripped"

        # Handle request normally...

    def should_trip(self):
        current_time = time.time()
        if self.last_failure_time and (current_time - self.last_failure_time) < self.reset_timeout:
            self.failures_count += 1
            return self.failures_count >= self.failure_threshold
        else:
            self.failures_count = 0
            self.last_failure_time = None
            return False

# Example usage
breaker = CircuitBreaker()
if breaker.handle_request() == "Circuit breaker tripped":
    # Fallback or handle failure...
else:
    # Proceed with normal operation...

#### Closing Summary
and Common Mistakes

Implementing these mechanisms will help mitigate the impact of retry storms during brownouts. Key points to remember include:

Exponential Backoff with Jitter helps spread out retries over time.
Retry Budget limits the number of retries per second to prevent overwhelming the system.
Circuit Breaker provides a safety net by failing fast when too many failures occur.

#### Common mistakes
to avoid:

Failing to introduce randomness in exponential backoff can lead to correlated retries and further stress on the system.
Setting the retry budget or failure threshold too low might result in frequent circuit breaker trips, while setting it too high could prolong recovery time unnecessarily.
Not resetting counters appropriately when a brownout period is over.

**Comparison:**

| Retry Attempt | Delay (Initial) | Delay with Jitter (Estimated) |
|---------------|-----------------|-------------------------------|
| 1             | 1s              | 1.5 - 2.5s                    |
| 2             | 2s              | 3.0 - 4.0s                    |
| 3             | 4s              | 6.0 - 8.0s                    |

**Code:**

```python
import random
import time

def calculate_backoff(retry_count, initial_delay=1):
    max_jitter = initial_delay * 0.5
    return (2 ** retry_count) * initial_delay + random.uniform(0, max_jitter)

class RetryBudget:
    def __init__(self, maximum_retries_per_second):
        self.maximum_retries_per_second = maximum_retries_per_second
        self.current_retries_this_second = 0

    def handle_retry(self, retry_count):
        if retry_count > self.maximum_retries_per_second:
            return "Retry budget exceeded"

        delay = calculate_backoff(retry_count)
        time.sleep(delay)
        # Perform database operation here...
        if operation_succeeded:
            reset_retry_budget()
        else:
            increment_retry_count()

class CircuitBreaker:
    def __init__(self, failure_threshold=100, reset_timeout=5 * 60):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.last_failure_time = None
        self.failures_count = 0

    def handle_request(self):
        if self.should_trip():
            return "Circuit breaker tripped"

        # Handle request normally...

    def should_trip(self):
        current_time = time.time()
        if self.last_failure_time and (current_time - self.last_failure_time) < self.reset_timeout:
            self.failures_count += 1
            return self.failures_count >= self.failure_threshold
        else:
            self.failures_count = 0
            self.last_failure_time = None
            return False

# Example usage
breaker = CircuitBreaker()
if breaker.handle_request() == "Circuit breaker tripped":
    # Fallback or handle failure...
else:
    # Proceed with normal operation...
```

**Example input / output:**

```text
Input:
- Initial delay = 1s, Max Jitter = 0.5 * initial delay
- Retry budget per second = 100
- Circuit breaker failure threshold = 100 failed writes in one minute

Output (Example):
Retry attempt 1: ~1s + random jitter
Retry attempt 2: ~3s - 5s
...

Circuit Breaker Tripped:
Input: More than 100 failed writes in one minute
Output: Fail fast and return error message
```

**Data:**

```json
{
  "database brownout": {
    "initial_delay": 1,
    "max_jitter": 0.5,
    "retry_budget_per_second": 100,
    "circuit_breaker_failure_threshold": 100,
    "circuit_breaker_reset_timeout": 300
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Handle Request
6. Should Circuit Breaker Trip?
7. Circuit Breaker Tripped
8. Calculate Backoff with Jitter
9. Perform Retry
10. Is Operation Succeeded?
11. Reset Retry Budget
12. Increment Retry Count

