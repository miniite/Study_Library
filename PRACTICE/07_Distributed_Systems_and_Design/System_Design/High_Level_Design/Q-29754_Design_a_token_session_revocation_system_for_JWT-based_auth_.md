---
type: "practice"
difficulty: "Deep"
---

# Design a token/session revocation system for JWT-based auth where a logout or a compromised-token event must invalidate access globally within seconds — how do you avoid a per-request DB check?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design an effective token revocation system that can invalidate JWTs globally in seconds upon logout or compromise, we need to leverage distributed cache systems like Redis and implement consistent hashing to avoid per-request database checks. This approach ensures quick invalidation while maintaining high availability and scalability.

#### Triaging the Question

The question asks us to design a robust token revocation mechanism for a JWT-based authentication system where the primary requirement is to invalidate access tokens globally within seconds. The key aspects we need to address are:

Scalability: Ensure that our solution can handle a large number of concurrent requests and users.
Performance: Minimize response times, ideally under 100ms or even faster for token revocation checks.

#### Cons
istency: Ensure that once a token is marked as invalid, all future request checks reflect this status instantly.
Fault Tolerance: The system should be resilient to cache failures and provide fallback mechanisms.

#### Assumptions
We are using JWTs with an expiration time (exp) claim for session management.
Tokens are stored in a distributed cache like Redis.
Users can log out or report compromised tokens, triggering revocation.
Our system handles a high volume of requests, potentially millions per second.

#### First Principles Solution
Token Storage: Store JWTs in a distributed cache (e.g., Redis) with an expiration time. This allows us to quickly invalidate tokens by removing them from the cache.
Revocation List: Implement a revocation list mechanism where compromised or expired tokens are stored and checked against during token validation.

#### Cons
istent Hashing for Token Lookup: Use consistent hashing to map JWTs to keys in Redis, ensuring that every token is mapped to the same key across different instances of our application.
Cache Invalidation Strategy: When a token is marked as invalid (e.g., on logout or reported compromise), invalidate it from the cache immediately.
Detailed Design
Token Storage and Revocation

Token Issuance: Upon user authentication, we issue JWTs with an expiration time (exp) claim. These tokens are stored in Redis under a key derived using consistent hashing.

PLAINTEXT
Key = hash(token)
Value = token

Revocation Handling:

On logout or compromised-token event: The system sends a command to invalidate the corresponding JWT from Redis.
This invalidation ensures that any subsequent requests with this token will fail due to cache absence.
Revocation List

For added security, maintain a revocation list of tokens that have been explicitly marked as invalid. This list is stored in a separate Redis key-value store.

PLAINTEXT
Key = revoked_tokens
Value = [token1, token2, ...]

During validation:

Check if the JWT exists in the cache (Redis) and then check against the revocation list.

#### Cons
istent Hashing

Use a consistent hashing algorithm to map tokens to Redis keys. This ensures that all instances of our application use the same mapping.

PLAINTEXT
Key = hash(token)

#### Example
For a token abc123, we might compute its key as follows:

key = sha256(abc123)
Store this key in Redis with the actual token value.
Cache Invalidation Strategy
When invalidating a token, use the consistent hash to derive the key and remove it from Redis.
PLAINTEXT
Key = hash(token)
redis.del(Key)

#### Implementation
Example
Yes
No
User Logs Out
Check Token Validity
Remove Token from Cache
Revoke Token via API
Token Found in Cache?
Mark as Revoked
Valid Token, Allow Access

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By leveraging a distributed cache and consistent hashing, we can quickly invalidate tokens globally. The revocation list ensures additional security by keeping track of explicitly marked invalid tokens.

#### Common Mistakes
Not using a distributed cache: Relying solely on the database for token validation will lead to performance issues under high load.
Poorly designed hash function: Using an ineffective or non-consistent hashing algorithm can result in suboptimal key distribution and increased conflicts.
Ignoring revocation lists: While the primary mechanism is caching, a revocation list adds another layer of security by allowing explicit marking of tokens as invalid.
Further Considerations
Rate Limiting: Implement rate limiting to prevent abuse from attackers trying to invalidate multiple tokens in rapid succession.
Fallback Mechanism: Ensure there are fallback mechanisms if Redis fails or experiences high latency, such as falling back to a database check with eventual consistency.

**Comparison:**

| Action | Cache Operation       | Revocation List Operation |
|--------|----------------------|--------------------------|
| Validate | `redis_client.exists(key)` | `redis_client.smembers('revoked_tokens')` |
| Revoke   | `redis_client.delete(key)`  | `redis_client.sadd('revoked_tokens', token.encode())` |

**Code:**

```python
# Example Python code for token validation and revocation

import hashlib
from redis import Redis

redis_client = Redis(host='localhost', port=6379)

def validate_token(token):
    # Check if the token exists in Redis cache
    key = hashlib.sha256(token.encode()).hexdigest()
    if not redis_client.exists(key):
        return False

    # Check against revocation list
    revoked_tokens = redis_client.smembers('revoked_tokens')
    if token.encode() in revoked_tokens:
        return False

    return True

def revoke_token(token):
    key = hashlib.sha256(token.encode()).hexdigest()
    redis_client.delete(key)
    redis_client.sadd('revoked_tokens', token.encode())
```

**Example input / output:**

```text
Input:
- Validate a token (e.g., abc123)
Output:
- True or False based on the result of validation

Example Dry Run:
Token: abc123
Cache Check: Token found in cache
Revocation List Check: Token not in revocation list
Result: True (valid)

Input:
- Revoke a token (e.g., def456)
Output:
- Token removed from Redis and added to revoked_tokens set

Example Dry Run:
Token: def456
Cache Removal: Key (def456) deleted from Redis
Revocation List Update: Token def456 added to revoked_tokens set
```

**Data:**

```json
{
  "token": "abc123",
  "revoked_tokens": ["def456", "ghi789"]
}
```

**Flow:**

1. Yes
2. No
3. User Logs Out
4. Check Token Validity
5. Remove Token from Cache
6. Revoke Token via API
7. Token Found in Cache?
8. Mark as Revoked
9. Valid Token, Allow Access

