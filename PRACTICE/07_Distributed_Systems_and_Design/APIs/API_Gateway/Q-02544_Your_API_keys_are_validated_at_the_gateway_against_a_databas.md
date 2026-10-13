---
type: "practice"
difficulty: "Deep"
---

# Your API keys are validated at the gateway against a database and that lookup is now the bottleneck; how do you cache validation, invalidate on key revocation, and bound staleness?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

To address the bottleneck in validating API keys at the gateway, we need to implement a caching strategy that not only accelerates lookups but also handles key revocations effectively while maintaining acceptable data freshness.

#### Triage

The core issue is improving performance by reducing database hits during API key validation. This involves:

Caching Validations: Storing validated keys in memory or a fast cache system.
Invalidate on Revocation: Ensuring that when an API key is revoked, its cached entry is invalidated.
Staleness Boundaries: Defining how old a cached key can be before it must be re-validated.

#### First Principles Solution
Caching Validations

Caching the validation results in memory (e.g., Redis) or a fast cache system allows quick lookups and reduces database load. For example, we could use Redis's SET command to store the API key as a boolean value indicating its validity:

SHELL
# Example: Store "true" for valid keys
$ redis-cli SET api_key:<key> true
Invalidate on Revocation

When an API key is revoked, the cached entry must be invalidated. Redis provides the DEL command to remove cache entries:

SHELL
# Example: Remove invalidation after revoking a key
$ redis-cli DEL api_key:<revoked_key>

To ensure this happens in real-time, we can use a pub/sub model where the database notification system sends messages to invalidate specific keys.

Staleness Boundaries

To handle staleness, we set an expiration time on cache entries. For instance, if the database does not update frequently (e.g., daily), a one-hour or two-hour TTL might be reasonable:

SHELL
# Example: Set an expiration for the key after 1 hour
$ redis-cli SETEX api_key:<key> 3600 true

#### Implementation
Steps

Database Notification Setup:

Configure your database to send a notification when an API key is revoked.
Use Redis' PUBLISH and SUBSCRIBE commands to handle real-time invalidation.

Cache Management:

When the gateway receives a request, check if the key exists in cache with a non-expired TTL.
If the key does not exist or has expired, fetch it from the database and update the cache.

API Key Revocation Handling:

Upon revoking an API key, send a message to Redis's pub/sub system.
The subscriber can then remove the invalid key entry in the cache.

Cache Hit/miss Logic:

If the key is found and still valid (within its TTL), return the result immediately.
Otherwise, validate from the database and update the cache as needed.

#### Closing Summary

By implementing a robust caching strategy with real-time invalidation and a staleness boundary, we can significantly reduce the bottleneck caused by frequent database lookups. This approach ensures that the API gateway remains performant even under high load.

#### Common Mistakes/Follow-Ups
Not Setting Proper Expiration Times: Failing to set an appropriate TTL for cache entries can lead to unnecessary database queries.
Real-Time Invalidation Misses: Not configuring real-time invalidation mechanisms means cached keys might not be updated in time, leading to security risks.
Cache Invalidation Lag: Delayed or misconfigured cache invalidation can introduce stale data issues.

Would you like a deeper dive into any specific aspect of this solution?

**Comparison:**

| Action | Cache Entry         | Expiration Time |
|--------|---------------------|----------------|
| Set    | api_key:<key> true  | 1 hour         |
| Publish| api_keys_revoked    | Real-time      |
| Remove | api_key:<revoked_key>| Immediate     |

**Code:**

```python
# Example Python script for real-time key revocation handling

import redis
from datetime import timedelta

r = redis.Redis(host='localhost', port=6379, db=0)

def revoke_api_key(key):
    # Publish revocation message to Redis pub/sub channel
    r.publish('api_keys_revoked', key)

# Example function to validate an API key with cache check
def validate_api_key(api_key):
    # Check if the key is in cache and within its TTL
    result = r.get(f'api_key:{api_key}')
    if result:
        return True  # Key is valid and not expired

    # Key not found or expired, fetch from database
    db_validation_result = fetch_from_database(api_key)

    # Update the cache with new validation result
    r.setex(f'api_key:{api_key}', timedelta(hours=1), str(db_validation_result))
    return db_validation_result

def fetch_from_database(api_key):
    # Placeholder for actual DB query logic
    pass
```

**Example input / output:**

```text
Input: Request to validate API key 'abc123'
- Cache check (Redis): Key found and not expired -> return True

Input: Revocation of API key 'def456'
- Publish revocation message: "api_keys_revoked:def456" sent via Redis pub/sub

Output: Key is invalidated from cache when subscription handles the message
```

**Data:**

```json
{
  "key": "api_key:<key>",
  "value": true,
  "expiration_time": 3600
}
```

**Flow:**

1. Hit
2. Miss
3. API Request
4. Cache Check
5. Return Validated Key
6. Fetch from DB
7. Update Cache
8. Return Validation Result
9. Real-time Revocation Handling

