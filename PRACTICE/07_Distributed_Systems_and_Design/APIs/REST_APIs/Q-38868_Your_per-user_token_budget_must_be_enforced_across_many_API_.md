---
type: "practice"
difficulty: "Expert"
---

# Your per-user token budget must be enforced across many API pods so no user can exceed their monthly LLM spend. How do you build a distributed quota in the REST hot path that counts tokens accurately without a synchronous store hit per request?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a per-user token budget across many API pods while ensuring accurate token counting, we need to design a distributed system that can handle high throughput and maintain consistency in a hot path scenario without relying on synchronous store hits. The solution involves leveraging a combination of stateless APIs with caching, event-driven architectures, and efficient data structures.

The core idea is to use an in-memory cache and a lightweight distributed synchronization mechanism to manage the token counts asynchronously while ensuring that the system can handle high request volumes efficiently. This approach balances performance with consistency and scalability.

**Comparison:**

```
User ID	Token Limit (tokens)	Current Tokens (tokens)
12345	1000	789
```

**Code:**

```python
# Example of a simple rate-limiter implementation using Redis for caching and locking

import redis
from contextlib import contextmanager

class TokenBudget:
    def __init__(self, user_id, token_limit):
        self.user_id = user_id
        self.token_limit = token_limit
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)

    @contextmanager
    def _lock(self):
        lock_key = f'budget_lock:{self.user_id}'
        acquired = False
        try:
            # Try to acquire a fair lock
            while not self.redis_client.set(lock_key, '1', nx=True, ex=5):
                pass
            acquired = True
            yield
        finally:
            if acquired and self.redis_client.delete(lock_key):
                print(f"Lock released for user {self.user_id}")

    def deduct_token(self, token_count):
        with self._lock():
            current_tokens = self.get_current_tokens()
            remaining_tokens = max(current_tokens - token_count, 0)
            if remaining_tokens >= 0:
                self.redis_client.setex(f'token_budget:{self.user_id}', 3600, remaining_tokens) # Set expiration to handle bursts
                return True
        return False

    def get_current_tokens(self):
        current_tokens = self.redis_client.get(f'token_budget:{self.user_id}')
        return int(current_tokens) if current_tokens else 0
```

**Example input / output:**

```text
Input:
- User ID: 12345
- Token Count: 10

Expected Output:
- True if tokens can be deducted, False otherwise.
```

**Data:**

```json
{
  "user_id": "12345",
  "token_limit": 1000,
  "current_tokens": 789
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Is Token Count Available inCache?
5. Fetch Current Tokens fromCache
6. Deduct Token Count
7. Save Updated Token Count toCache
8. Return Success/Failure
9. Lock User ID for Update
10. Fetch Current Tokens fromPersistent Store

