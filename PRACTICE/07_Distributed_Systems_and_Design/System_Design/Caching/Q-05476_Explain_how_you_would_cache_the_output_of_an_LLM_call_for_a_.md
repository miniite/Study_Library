---
type: "practice"
difficulty: "Deep"
---

# Explain how you would cache the output of an LLM call for a support-bot feature, deciding what is safe to cache and what would produce wrong or stale answers.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To cache the output of an LLM (Large Language Model) call for a support-bot feature, we need to carefully consider several factors to ensure that cached responses are safe, accurate, and up-to-date. The key steps involve understanding the nature of the questions asked by users, identifying safe caching scenarios, setting appropriate cache expiration policies, handling concurrency, and implementing mechanisms to invalidate or update the cache.

The core idea is to balance the benefits of caching (reduced latency and cost) against potential risks such as outdated information, privacy issues, and compliance concerns. By following a structured approach, we can develop a robust caching strategy that minimizes these risks while maximizing efficiency.

**Comparison:**

```
Scenario	Safe to Cache?	Reasoning
Common troubleshooting questions	Yes	Typically don't change frequently.
Legal or compliance-related queries	No	Could become outdated, privacy issues.
User-specific questions	Maybe	Needs careful consideration; could be sensitive and vary over time.
```

**Code:**

```python
import hashlib

def generate_cache_key(user_id, question):
    return hashlib.md5(f"{user_id}:{question}".encode()).hexdigest()

class LLMCache:
    def __init__(self, cache_timeout=3600):  # Default to 1 hour
        self.cache = {}
        self.expirations = {}  # Maps key to expiration time

    def get_response(self, user_id, question):
        cache_key = generate_cache_key(user_id, question)
        if cache_key in self.cache:
            response, expires_at = self.cache[cache_key]
            if expires_at > time.time():
                return response
            else:
                del self.cache[cache_key]  # Remove expired entries

        # Fetch from LLM and store the result with a timestamp
        response = fetch_llm_response(question)
        self.cache[cache_key] = (response, time.time() + cache_timeout)
        self.expirations[cache_key] = time.time() + cache_timeout
        return response

    def invalidate_cache(self, user_id, question):
        cache_key = generate_cache_key(user_id, question)
        if cache_key in self.cache:
            del self.cache[cache_key]
            if cache_key in self.expirations:
                del self.expirations[cache_key]

The above Python code implements a basic caching mechanism using a dictionary to store responses and their expiration times. The get_response method checks for an existing cached response, retrieves it if fresh, or fetches from the LLM otherwise. The invalidate_cache method allows us to manually remove entries.
```

**Example input / output:**

```text
Input:
- User ID: 12345
- Question: "How can I troubleshoot my connection issues?"

Output:
- Response (cached): "This is a cached response from the LLM."
- Expiration Time: 1633072800

(When time > 1633072800, the cache will be invalidated and re-fetched)
```

**Data:**

```json
{
  "cache_key": "8d961c9e68e7549f782fe05b0b4a22c",
  "response": "This is a cached response from the LLM.",
  "expires_at": 1633072800
}
```

**Flow:**

1. Yes
2. No
3. User asks a question
4. Is the question safe to cache?
5. Get cached response
6. Fetch from LLM, then cache
7. Cache expires
8. Invalidate and fetch again

