---
type: "practice"
difficulty: "Deep"
---

# Your gateway's JWKS cache refreshes on a timer but the IdP rotated keys off-schedule and valid tokens started 401ing; how do you implement refresh-on-miss with backoff so an unknown kid triggers a controlled JWKS re-fetch?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle the scenario where your API Gateway's JWKS cache refreshes on a timer but the IdP rotates keys off-schedule, resulting in 401 errors due to expired tokens, we can implement a "refresh-on-miss" mechanism with backoff. This ensures that an unknown key triggers a controlled re-fetch of the JWKS. The core idea is to maintain a robust state machine for managing cache freshness and retries.

First, let's break down what the question is asking:

Key Rotations: The IdP rotates keys, but not necessarily on a fixed schedule.
Token Validation Errors: Tokens are being invalidated by unexpected key rotations.
JWKS Cache: A timer-based cache refresh is insufficient during key rotation events.
Backoff Strategy: Implementing retries to avoid flooding the IdP with requests.
Triaging the Problem

We need to ensure that when a token fails due to an unknown key, we trigger a controlled re-fetch of the JWKS. The mechanism should handle this in a way that minimizes the number of requests sent to the IdP and ensures smooth operation under unexpected conditions.

#### Assumptions
Token Validation: Tokens are validated against the JWKS.
JWKS Cache: The cache has an expiry time.
IdP Response Time: The IdP can handle a reasonable number of requests.
Retry Mechanism: We need to implement exponential backoff to avoid overwhelming the IdP.

#### First-Principles Solution
State Machine for JWKS Cache:
Define states: Cached, UnknownKey, Fetching.
Backoff Strategy:
Implement an exponential backoff with jitter.
Event-Driven Refresh:
Trigger a re-fetch when a token fails validation due to an unknown key.
Detailed Implementation
State Machine for JWKS Cache
PYTHON
class JwkCache:
    def __init__(self, jwks_url, cache_expiry):
        self.jwks = None
        self.expiry_time = 0
        self.url = jwks_url
        self.cache_expiry = cache_expiry

    def get_jwks(self):
        if time.time() > self.expiry_time:
            return self.fetch_jwks()
        return self.jwks

    def fetch_jwks(self, retry_count=0):
        try:
            response = requests.get(self.url)
            jwks = response.json()
            self.jwks = jwks
            self.expiry_time = time.time() + self.cache_expiry
            return jwks
        except Exception as e:
            if retry_count < 5:  # Maximum retries
                sleep_time = min(2 ** retry_count, 30) + random.random()
                print(f"Failed to fetch JWKS, retrying in {sleep_time} seconds")
                time.sleep(sleep_time)
                return self.fetch_jwks(retry_count + 1)
            else:
                raise e
Token Validation and Cache Management
PYTHON
def validate_token(token):
    jwk_cache = JwkCache(jwks_url, cache_expiry=3600)  # URL and cache expiry time in seconds

    try:
        decoded_token = jwt.decode(token, options={"verify_signature": True}, algorithms=["RS256"], keys=jwk_cache.get_jwks())
        return decoded_token
    except Exception as e:
        if "unknown key" in str(e):
            print("Token validation failed due to unknown key. Triggering re-fetch.")
            jwk_cache.fetch_jwks()
            # Retry token validation after refresh
            try:
                return jwt.decode(token, options={"verify_signature": True}, algorithms=["RS256"], keys=jwk_cache.get_jwks())
            except Exception as e:
                print(f"Failed to validate token: {e}")
        else:
            raise e

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The solution involves using a state machine for JWKS caching and implementing an exponential backoff strategy. This ensures that when tokens fail due to unknown keys, the cache is refreshed in a controlled manner.

#### Common Mistakes
Failing to implement retries and backoff, leading to unnecessary IdP requests.
Not handling edge cases where token validation fails for reasons other than key mismatches.
Overlooking state transitions and ensuring they are handled correctly.

#### Follow-Ups
What happens if the IdP is down or unresponsive during the refresh?
Implement a circuit breaker pattern to manage such scenarios.
How do you handle different token types (e.g., access, refresh)?
Differentiate validation logic based on token type and cache handling.
Are there any performance implications of this approach?

#### Cons
ider the impact on API Gateway's performance during key rotations.

**Comparison:**

```
State	Action
Cached	Use current cached JWKS.
UnknownKey	Trigger re-fetch and retry validation.
Fetching	Fetch new JWKS and update cache.
```

**Code:**

```python
import time
import requests
import jwt
from random import random

class JwkCache:
    def __init__(self, jwks_url, cache_expiry):
        self.jwks = None
        self.expiry_time = 0
        self.url = jwks_url
        self.cache_expiry = cache_expiry

    def get_jwks(self):
        if time.time() > self.expiry_time:
            return self.fetch_jwks()
        return self.jwks

    def fetch_jwks(self, retry_count=0):
        try:
            response = requests.get(self.url)
            jwks = response.json()
            self.jwks = jwks
            self.expiry_time = time.time() + self.cache_expiry
            return jwks
        except Exception as e:
            if retry_count < 5:  # Maximum retries
                sleep_time = min(2 ** retry_count, 30) + random.random()
                print(f"Failed to fetch JWKS, retrying in {sleep_time} seconds")
                time.sleep(sleep_time)
                return self.fetch_jwks(retry_count + 1)
            else:
                raise e

def validate_token(token):
    jwk_cache = JwkCache(jwks_url="https://example.com/jwks", cache_expiry=3600)  # URL and cache expiry time in seconds

    try:
        decoded_token = jwt.decode(token, options={"verify_signature": True}, algorithms=["RS256"], keys=jwk_cache.get_jwks())
        return decoded_token
    except Exception as e:
        if "unknown key" in str(e):
            print("Token validation failed due to unknown key. Triggering re-fetch.")
            jwk_cache.fetch_jwks()
            # Retry token validation after refresh
            try:
                return jwt.decode(token, options={"verify_signature": True}, algorithms=["RS256"], keys=jwk_cache.get_jwks())
            except Exception as e:
                print(f"Failed to validate token: {e}")
        else:
            raise e
```

**Example input / output:**

```text
token = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
try:
    decoded_token = validate_token(token)
    print(f"Decoded token: {decoded_token}")
except Exception as e:
    print(f"Error decoding token: {e}")
```

**Data:**

```json
{
  "jwks_url": "https://example.com/jwks",
  "cache_expiry": 3600,
  "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

**Flow:**

1. Yes
2. No
3. Expiry Time <= CurrentTime
4. Expiry Time > Current Time
5. Yes
6. No
7. Token Validation
8. Valid Token?
9. Return Decoded Token
10. Check Cache Expiry Time
11. Fetch New JWKS and RetryValidation
12. Use Cached JWKS and RetryValidation with Backoff
13. Handle Error or Retries
14. Retry Count < Max Retries?
15. Sleep for Jittered Time, ThenFetch Again

