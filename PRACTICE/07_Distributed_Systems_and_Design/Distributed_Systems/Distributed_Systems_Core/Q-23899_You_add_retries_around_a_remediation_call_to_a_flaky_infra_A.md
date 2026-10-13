---
type: "practice"
difficulty: "Intermediate"
---

# You add retries around a remediation call to a flaky infra API, but a retry after an ambiguous timeout scaled a group twice. Explain which errors are safely retryable, why timeouts on the scale call are ambiguous, and how you'd make the ambiguous ones safe.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with retries in a distributed system where a flaky infrastructure API is causing issues. The goal is to safely manage retries while ensuring that scale operations are handled correctly to avoid unintended duplication. Let's break down the problem and explore the solution step-by-step.

The key challenge here is understanding which errors can be safely retried, why timeouts on the scale call might be ambiguous, and how we can handle these ambiguities effectively.

**Comparison:**

```
Error Type	Retriability	Reason
Connection Timeout	Safe to retry	Typically transient network issues.
Ambiguous Timeout	Safe to retry with caution	Scales the group unexpectedly. Needs handling carefully.
Other Errors (e.g., Resource Limit Exceeded)	Not safe to retry	Could lead to resource contention and instability.
```

**Code:**

```python
# Example code snippet to illustrate retry logic
from time import sleep

def retry_with_exponential_backoff(attempts, initial_delay, factor=2):
    attempt = 0
    while attempt < attempts:
        try:
            return call_infra_api()
        except FlakyAPIError as e:
            if "ambiguous timeout" in str(e):
                print(f"Ambiguous timeout detected. Retrying after {initial_delay} seconds...")
                sleep(initial_delay)
                initial_delay *= factor
            else:
                raise  # Rethrow the error for other types of failures

def call_infra_api():
    # Simulated API call with potential flakiness
    pass
```

**Example input / output:**

```
Input: A call_infra_api function that might throw a FlakyAPIError.
Output: Controlled retry logic based on error types.

Example scenario:

PYTHON
Copy
def call_infra_api():
    # Simulated API call with potential flakiness
    if random.random() < 0.3:  # 30% chance of failure
        raise FlakyAPIError("Connection timed out")
    elif random.random() < 0.1:  # 10% chance of an ambiguous timeout
        raise FlakyAPIError("Ambiguous timeout detected")

# Retry mechanism
retry_with_exponential_backoff(attempts=3, initial_delay=5)
```

**Data:**

```json
{
  "api_call_attempts": 3,
  "initial_delay_seconds": 5,
  "exponential_factor": 2
}
```

**Flow:**

1. Connection Timeout
2. Ambiguous Timeout
3. Other Errors
4. Call Infra API
5. Error?
6. Retry with Exponential Backoff
7. Log Error, Alert Team
8. Raise Error, No Retry
9. Retried Call
10. Handle Scale Event
11. Scale Event Handled
12. Scale Operation Resolved
13. Operation Retried or Failed
14. Scale Done

