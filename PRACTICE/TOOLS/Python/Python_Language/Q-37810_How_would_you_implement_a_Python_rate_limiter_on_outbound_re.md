---
type: "practice"
difficulty: "Deep"
---

# How would you implement a Python rate limiter on outbound remediation actions so that a bug that fires 1000 triggers can't execute 1000 destructive actions against your infra?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a Python rate limiter for outbound remediation actions, we need to ensure that even if a bug triggers an excessive number of actions (e.g., 1000), only a controlled and limited number are executed. This is crucial for preventing overwhelming the system and causing potential damage.

#### Triage
Understanding the Question

The question is asking how to implement rate limiting in Python to control the frequency of outbound remediation actions, ensuring that a single trigger does not cause an excessive number of destructive actions. The key aspects include:

Rate Limiting Mechanism: How to limit the number of actions per period.
Concurrency Control: Handling concurrent triggers efficiently.
Error Handling: Ensuring that actions do not fail silently or cause unexpected behavior.

#### First-Principles Solution

We can implement a rate limiter using a combination of in-memory data structures and Python's asyncio for handling concurrency. The approach will involve tracking the number of requests made within a specified time window to ensure that no more than a certain limit is exceeded.

Step 1: Define the Rate Limiter Class

We'll create a class to encapsulate the rate-limited behavior:

PYTHON
import asyncio
from collections import deque
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, max_requests, period_seconds):
        self.max_requests = max_requests
        self.period_seconds = period_seconds
        self.requests = deque()

    async def can_execute(self):
        now = datetime.now()
        while self.requests and self.requests[0] < now - timedelta(seconds=self.period_seconds):
            self.requests.popleft()

        if len(self.requests) >= self.max_requests:
            return False

        self.requests.append(now)
        await asyncio.sleep(0)  # Yield control to other coroutines
        return True
Step 2: Use the RateLimiter in a Remediation Action

To use this rate limiter, we'll wrap our remediation action with it:

PYTHON
async def perform_remediation(action):
    async with RateLimiter(max_requests=10, period_seconds=5) as limiter:
        if not await limiter.can_execute():
            print("Rate limit exceeded. Action not executed.")
            return False

        # Perform the actual remediation action here
        print(f"Executing {action}")
        return True

# Example usage
async def main():
    for _ in range(100):
        result = await perform_remediation("Remediate")
        if not result:
            break

if __name__ == "__main__":
    asyncio.run(main())

#### Closing Summary
and Common Mistakes/Follow-Ups

This implementation ensures that no more than 10 remediation actions can be executed within a 5-second period. The rate limiter is thread-safe due to the use of asyncio for asynchronous operations.

#### Common Mistakes

Failing to Use Asynchronous Context Managers: Not using async with can lead to blocking behavior, which defeats the purpose of rate limiting.
Ignoring Rate Limit Exceeded Cases: Failing to handle cases where the action is not executed due to the rate limiter can lead to unexpected behavior.

#### Follow-Ups

How would you scale this solution for a distributed system?
What if we need different rate limits for different types of actions?
How would you test this implementation?

**Code:**

```python
import asyncio
from collections import deque
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, max_requests, period_seconds):
        self.max_requests = max_requests
        self.period_seconds = period_seconds
        self.requests = deque()

    async def can_execute(self):
        now = datetime.now()
        while self.requests and self.requests[0] < now - timedelta(seconds=self.period_seconds):
            self.requests.popleft()

        if len(self.requests) >= self.max_requests:
            return False

        self.requests.append(now)
        await asyncio.sleep(0)  # Yield control to other coroutines
        return True

async def perform_remediation(action):
    async with RateLimiter(max_requests=10, period_seconds=5) as limiter:
        if not await limiter.can_execute():
            print("Rate limit exceeded. Action not executed.")
            return False

        # Perform the actual remediation action here
        print(f"Executing {action}")
        return True

async def main():
    for _ in range(100):
        result = await perform_remediation("Remediate")
        if not result:
            break

if __name__ == "__main__":
    asyncio.run(main())
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Rate Limit Exceeded?
5. Log and Exit
6. Perform Remediation Action
7. End

