---
type: "practice"
difficulty: "Deep"
---

# Your gateway offloads JWT validation from every microservice; walk through how you verify tokens at the edge, cache JWKS, and handle key rotation without dropping valid requests during a rotation window.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this scenario, we're discussing an API Gateway that offloads JWT validation from microservices. The key aspects to cover are token verification at the edge, caching of JWKS (JSON Web Key Set) for performance optimization, and handling key rotation while ensuring uninterrupted service.

#### Triaging the Question

The interviewer is probing our understanding of secure APIs, caching strategies, key management practices, and robustness during operational changes. We need to ensure that valid requests are not dropped during key rotation periods. Our solution should balance security with performance and maintain high availability.

Key Assumptions:
JWT Validation: The gateway validates JWT tokens against a public key set.
Key Rotation: Periodic updates in the public key set (JWKS) require handling without affecting existing valid sessions.

#### Performance Considerations
Caching is used to reduce the load on the key server.

#### First-Principles Solution

Token Verification at the Edge:

The gateway receives a JWT, extracts its header and payload.
Validates the token's signature using the public keys from the JWKS provided by the key server.
Checks if the token is valid within its expiration period.

Caching JWKS:

Cache the JWKS to avoid frequent calls to the key server for every JWT validation.
Use a cache with TTL (Time To Live) to balance between freshness and performance.

Handling Key Rotation:

Maintain multiple versions of the public keys in the JWKS.
During rotation, add new keys while keeping old ones valid until all sessions using them expire.
Implement a rolling update strategy where both sets are valid for a transition period to prevent dropping valid requests.
Step-by-Step Solution:

Token Verification:

PYTHON
def validate_jwt(token: str) -> bool:
    header, payload = extract_header_and_payload(token)
    jwks = get_jwks_from_cache()
    key_id = header['kid']
    signing_key = find_signing_key(jwks, key_id)
    if not verify_signature(payload, signing_key):
        return False
    if not is_token_valid_within_expiration_window(payload):
        return False
    return True

Caching JWKS:

Use an in-memory cache like Redis to store the JWKS.
PYTHON
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def get_jwks_from_cache():
    jwks_json = r.get('jwks')
    if jwks_json:
        return json.loads(jwks_json)
    # Fetch JWKS from the key server and update cache
    ...

def update_jwks_in_cache(new_jwks):
    r.set('jwks', json.dumps(new_jwks), ex=3600)  # Cache TTL of 1 hour

Handling Key Rotation:

Introduce a rolling key update strategy.
PYTHON
def handle_key_rotation(old_keys: List[dict], new_keys: List[dict]):
    transition_window = 24 * 60 * 60  # Transition window of 1 day

    def validate_jwt_with_keys(token: str, keys) -> bool:
        header, payload = extract_header_and_payload(token)
        key_id = header['kid']
        signing_key = find_signing_key(keys, key_id)
        if not verify_signature(payload, signing_key):
            return False
        if not is_token_valid_within_expiration_window(payload):
            return False
        return True

    # Start with old keys and transition to new keys over the window
    for i in range(transition_window):
        time.sleep(1)  # Simulate gradual key rotation
        if validate_jwt_with_keys(token, old_keys + new_keys):
            return True
    return validate_jwt_with_keys(token, new_keys)

#### Closing Summary

The API Gateway verifies JWTs at the edge using a cached JWKS, ensuring performance optimization and minimal load on the key server. During key rotation, multiple versions of keys are maintained to avoid dropping valid requests during the transition period. This approach ensures robust security while maintaining service availability.

#### Common Mistakes/Follow-Ups
Failing to Cache JWKS: Caching can significantly impact performance.
Not Handling Key Rotation Gracefully: Dropping valid tokens during rotation can lead to downtime or user frustration.
Ignoring Token Expiration Checks: Incorrect expiration checks can result in token validation failures.

Are there any follow-ups you’d like to explore further? For example, how would you secure the JWKS cache itself, or what monitoring and logging mechanisms would you implement?

**Comparison:**

```
Key	Description
kid	Key ID
kty	Key Type
n	Public Modulus
e	Public Exponent
Transition State	Time	Action
Pre-Rotation	0-12 hours	Use old keys only
During Rotation	13-24 hours	Use both keys
Post-Rotation	>24 hours	Use new keys only
```

**Code:**

```python
def validate_jwt(token: str) -> bool:
    header, payload = extract_header_and_payload(token)
    jwks = get_jwks_from_cache()
    key_id = header['kid']
    signing_key = find_signing_key(jwks, key_id)
    if not verify_signature(payload, signing_key):
        return False
    if not is_token_valid_within_expiration_window(payload):
        return False
    return True

def get_jwks_from_cache():
    jwks_json = r.get('jwks')
    if jwks_json:
        return json.loads(jwks_json)
    # Fetch JWKS from the key server and update cache
    ...

def update_jwks_in_cache(new_jwks):
    r.set('jwks', json.dumps(new_jwks), ex=3600)  # Cache TTL of 1 hour

def handle_key_rotation(old_keys: List[dict], new_keys: List[dict]):
    transition_window = 24 * 60 * 60  # Transition window of 1 day

    def validate_jwt_with_keys(token: str, keys) -> bool:
        header, payload = extract_header_and_payload(token)
        key_id = header['kid']
        signing_key = find_signing_key(keys, key_id)
        if not verify_signature(payload, signing_key):
            return False
        if not is_token_valid_within_expiration_window(payload):
            return False
        return True

    # Start with old keys and transition to new keys over the window
    for i in range(transition_window):
        time.sleep(1)  # Simulate gradual key rotation
        if validate_jwt_with_keys(token, old_keys + new_keys):
            return True
    return validate_jwt_with_keys(token, new_keys)
```

**Example input / output:**

```text
Input:
- JWT token with valid signature and not expired.

Output:
- True (if the token is valid)
- False (if the token is invalid or expired)

Example:
- Token: {"header": {"kid": "old_key_id"}, "payload": {"exp": 1689320400, ...}, "signature": "..."}
```

**Data:**

```json
{
  "jwks": {
    "keys": [
      {"kid": "old_key_id", "kty": "RSA", "n": "...", "e": "..."},
      {"kid": "new_key_id", "kty": "RSA", "n": "...", "e": "..."}
    ]
  }
}
```

**Flow:**

1. Valid
2. Expired
3. Receive JWT
4. Extract Header & Payload
5. Fetch JWKS from Cache
6. Find Signing Key
7. Verify Signature
8. Check Expiration
9. Return True
10. Return False

