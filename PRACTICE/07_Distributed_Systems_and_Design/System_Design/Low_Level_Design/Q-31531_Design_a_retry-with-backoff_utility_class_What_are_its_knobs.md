---
type: "practice"
difficulty: "Intermediate"
---

# Design a retry-with-backoff utility class. What are its knobs and how do you cap total attempts?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a retry-with-backoff utility class, we need to focus on two primary aspects: managing the number of retries and implementing an exponential backoff strategy. The core idea is to create a class that can handle retries with delays between attempts, which helps in dealing with transient failures gracefully.

The utility class will have several knobs such as maxRetries, initialDelay, multiplier, and maxDelay to control the retry behavior. We'll also need to ensure that these knobs are appropriately capped to prevent infinite loops or excessive delay accumulation.

Let's break down the design step by step from first principles.

**Comparison:**

```
Parameter	Description
maxRetries	Maximum number of retries allowed.
initialDelay	Initial delay between attempts, in milliseconds.
multiplier	Multiplier for increasing the delay with each retry.
maxDelay	Maximum delay allowed per attempt, to avoid infinite delays.
```

**Code:**

```python
class RetryWithBackoff:
    def __init__(self, maxRetries=5, initialDelay=100, multiplier=2, maxDelay=3000):
        self.maxRetries = maxRetries
        self.initialDelay = initialDelay  # in milliseconds
        self.multiplier = multiplier
        self.maxDelay = maxDelay  # in milliseconds

    def retry(self, operation):
        currentDelay = self.initialDelay
        for attempt in range(1, self.maxRetries + 2):  # Including the initial attempt
            try:
                return operation()
            except Exception as e:
                if attempt > self.maxRetries:
                    raise RetryExceededLimitException(f"Max retries exceeded: {e}")
                else:
                    print(f"Attempt {attempt} failed. Retrying in {currentDelay / 1000} seconds...")
                    time.sleep(currentDelay / 1000)
                    currentDelay = min(currentDelay * self.multiplier, self.maxDelay)

class RetryExceededLimitException(Exception):
    pass
```

**Example input / output:**

```text
# Example usage
def test_function():
    # Simulate a random failure
    if random.random() < 0.7:
        raise Exception("Test error")
    print("Operation succeeded")

retry_util = RetryWithBackoff(maxRetries=3, initialDelay=500, multiplier=2, maxDelay=1000)
try:
    result = retry_util.retry(test_function)
except RetryExceededLimitException as e:
    print(e)

# Output might be something like:
# Attempt 1 failed. Retrying in 0.5 seconds...
# Attempt 2 failed. Retrying in 1 second...
# Operation succeeded
```

**Data:**

```text
The class `RetryWithBackoff` has the following parameters:
- `maxRetries`: Maximum number of retries.
- `initialDelay`: Initial delay between attempts in milliseconds.
- `multiplier`: Multiplier for increasing the delay with each retry.
- `maxDelay`: Maximum delay allowed in milliseconds, to avoid infinite delays.
```

**Flow:**

1. Yes
2. Exception
3. No
4. Start
5. Attempt <= maxRetries?
6. Execute Operation
7. Check for Exception
8. Increase Delay and Retry
9. Delay = min(currentDelay *multiplier, maxDelay)
10. RaiseRetryExceededLimitException

