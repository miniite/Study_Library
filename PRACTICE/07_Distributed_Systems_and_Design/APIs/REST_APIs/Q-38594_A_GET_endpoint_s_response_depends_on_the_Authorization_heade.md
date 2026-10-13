---
type: "practice"
difficulty: "Expert"
---

# A GET endpoint's response depends on the Authorization header, but a shared CDN cached one user's response and served it to others. What went wrong with your cache keys and how do you fix it safely?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, the root cause of the issue lies in improper caching strategies, specifically related to the use of Authorization headers which are inherently sensitive and user-specific. To address this problem, we need to ensure that our cache keys do not include dynamic or variable parts such as authentication tokens. Here’s a step-by-step solution to fix the caching safely:

#### Triage

The question is probing our understanding of HTTP caching mechanisms and their interaction with authentication headers. We are expected to explain why using Authorization headers can lead to issues, how this was incorrectly cached, and provide a safe way to reconfigure our cache keys.

Core Solution & Key Idea

We need to ensure that our cache keys do not include the Authorization header or any other user-specific information. Instead, we should use unique identifiers such as resource URLs or hashes based on query parameters. This will prevent caching issues related to sensitive data and ensure proper response distribution.

Step-by-Step Solution from First Principles

Understand the Issue:

The GET endpoint relies on an Authorization header for user authentication.
A shared CDN cached a specific user's response, leading to incorrect responses being served to other users with different credentials.

Identify the Problem:

Cache keys should not include dynamic or variable parts like Authorization headers because these can vary between users and invalidate cache entries unintentionally.
The CDN cached based on a key that included user-specific data, which led to incorrect caching behavior.

Designing a Safe Caching Strategy:

Use unique identifiers in cache keys that do not include sensitive information like Authorization headers.
Ensure the cache key is consistent for requests with identical parameters but different users.

#### Implementation
Details:

For this GET endpoint, instead of using the full URL or query string, use a hash based on the query parameters (e.g., path + query) to create a unique and stable identifier.

#### Example
Cache Key Generation:
PYTHON
import hashlib

def generate_cache_key(request):
    # Extract relevant parts from request
    path = request.path
    params = {k: v for k, v in request.GET.items() if k != 'Authorization'}

    key_parts = [path] + list(params.items())
    key_str = '&'.join([f'{k}={v}' for k, v in sorted(key_parts)])

    return hashlib.sha256(key_str.encode()).hexdigest()
Store the cache entries using this generated key to ensure consistent responses.

#### Closing Summary

By ensuring that our cache keys do not include user-specific or dynamic elements like Authorization headers, we can prevent caching issues and ensure proper response distribution. This approach maintains the integrity of our caching mechanism while respecting security best practices.

#### Common Mistakes
& Follow-Ups

Mistake 1: Including sensitive information in cache keys.

#### Follow-Up
Always exclude user-specific or dynamic data from cache keys to avoid invalidating caches unintentionally.

Mistake 2: Failing to update the cache key generation logic when request parameters change.

#### Follow-Up
Regularly review and test cache key generation logic as part of feature updates and refactoring efforts.

**Code:**

```python
import hashlib

def generate_cache_key(request):
    path = request.path
    params = {k: v for k, v in request.GET.items() if k != 'Authorization'}

    key_parts = [path] + list(params.items())
    key_str = '&'.join([f'{k}={v}' for k, v in sorted(key_parts)])

    return hashlib.sha256(key_str.encode()).hexdigest()
```

