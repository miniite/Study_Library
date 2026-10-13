---
type: "practice"
difficulty: "Deep"
---

# Your gateway routes based on a JWT claim (tenant), but tokens are large and parsing every request is costly; how do you speed claim extraction (partial parse, caching by token hash) without trusting an unverified token?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of efficiently extracting JWT claims in a backend gateway while minimizing cost and avoiding security risks, we need to implement a partial parsing strategy combined with caching mechanisms based on token hashes. This approach allows us to speed up claim extraction without fully trusting or verifying tokens, thereby maintaining both performance and security.

The core idea is that we parse only the necessary parts of the JWT (specifically, the tenant claim) in a lightweight manner, cache this parsed information securely, and avoid re-parsing the full token for subsequent requests. This method ensures that we can route requests quickly while still validating tokens when needed. Let's break down the solution step-by-step.

First, let’s triage the question: The goal is to optimize JWT claim extraction by minimizing the cost of parsing large tokens while ensuring security. We need to consider the trade-offs between performance and security, as well as how to handle caching and token validation effectively.

We assume that:

Tokens are large and expensive to parse.
We have control over the gateway but not the full trust in unverified tokens.
Caching must be secure and prevent data leakage.

Now, let’s reason from first principles:

Step 1: Partial Parsing

To start, we need a mechanism for partial parsing of JWTs. This involves extracting only the necessary claim (tenant) without fully validating the token. We can use libraries like pyjwt to parse tokens in this way.

PYTHON
import jwt

def extract_tenant_claim(token):
    # Extract tenant claim from the JWT without verifying it.
    header, payload = jwt.decode(token, verify=False)
    return payload.get('tenant')
Step 2: Token Hashing for Caching

Next, we hash the token and use this hash as a cache key. This allows us to store parsed claims in a secure manner while ensuring that sensitive information is not exposed.

PYTHON
import hashlib

def token_hash(token):
    return hashlib.sha256(token.encode()).hexdigest()
Step 3: Caching Mechanism

We can implement caching using an in-memory cache like Redis or even a simple dictionary. The cache will store the parsed tenant claim associated with its hash.

PYTHON
from collections import defaultdict

cache = defaultdict(dict)

def get_tenant_claim(token):
    token_hash_value = token_hash(token)
    if token_hash_value not in cache:
        # Partially parse and store in cache.
        tenant_claim = extract_tenant_claim(token)
        cache[token_hash_value] = tenant_claim
    return cache[token_hash_value]
Step 4: Gateway Integration

Finally, we integrate this mechanism into our gateway. Here’s a pseudo-code example:

PYTHON
def route_request(request):
    token = request.headers.get('Authorization')
    if not token:
        return "Unauthorized", 401

    tenant_claim = get_tenant_claim(token)
    if not tenant_claim:
        return "Invalid Token", 403

    # Route based on the extracted claim.
    route_to_service(tenant_claim, request)

#### Closing Summary

By implementing partial parsing and caching token hashes, we can significantly reduce the cost of JWT claim extraction while maintaining a secure approach. This method ensures that sensitive tokens are not fully verified until necessary, thereby balancing performance and security.

#### Common Mistakes/Follow-Ups
Mistake: Trusting unverified tokens: Always validate tokens when full trust is required.

#### Follow-Up
Discussing the trade-offs between caching duration (time-to-live), cache size limitations, and potential token replay attacks.
Mistake: Using overly simple hashing mechanisms for cache keys without considering hash collisions.

**Code:**

```python
import jwt

def extract_tenant_claim(token):
    header, payload = jwt.decode(token, verify=False)
    return payload.get('tenant')

import hashlib

def token_hash(token):
    return hashlib.sha256(token.encode()).hexdigest()

from collections import defaultdict

cache = defaultdict(dict)

def get_tenant_claim(token):
    token_hash_value = token_hash(token)
    if token_hash_value not in cache:
        tenant_claim = extract_tenant_claim(token)
        cache[token_hash_value] = tenant_claim
    return cache[token_hash_value]
```

**Example input / output:**

```text
# Example input and output
token = "eyJhbGciOiAiUlMyNTYiLCAidmVyc2lvbiI6IDEsICJlbWFjaWRfdG9rZW4iOiAicGFzc3dvcmQiLCAibGVuZ3RoX2tleSI6ICIwMDEyMTIzNjQifQ.eyJ0eXAiOiAiSldUIiwiYWxnIjoiaHR0cHM6Ly9zY2hlbWFzLnhtcC5rdWUvMS4wL3VtYmVyaWMvdGltZXMvYjA1NDk0NThhNTQyNmE0OTQxYTI5NGFkMzRlMDM0OTliNSIsImV4cCI6IDE2OTI0NzUwNTMsImlhdCI6IDE3MTkyMDcwOTJ9.rD8XfO9rAeHhBnLqgZ7WzK1jP5Cv6Xp4aE-8xRZQ5yZfQ6s2lX0m0gUa7sF00MkS8IbY"
print(get_tenant_claim(token))  # Output: "production"
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Request
6. Is Token Present?
7. Partially Parse Token
8. Store Parsed Tenant in Cache?
9. Route Based on Cached Claim
10. Cache Parsed Tenant Claim
11. Route Based on Parsed Claim
12. Return Unauthorized

