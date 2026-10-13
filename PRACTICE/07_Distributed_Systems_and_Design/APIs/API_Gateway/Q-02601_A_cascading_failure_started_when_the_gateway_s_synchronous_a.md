---
type: "practice"
difficulty: "Expert"
---

# A cascading failure started when the gateway's synchronous auth-service dependency slowed, backing up gateway threads and taking down all APIs; retrospect this and redesign the dependency so the gateway degrades gracefully rather than coupling every API's availability to one service.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a cascading failure scenario, the primary concern is ensuring that critical services can continue operating even when dependent services are slow or unavailable. For an API Gateway designed with synchronous auth-service dependencies, this results in a single point of failure where all APIs become inaccessible once the auth service slows down. To redesign the gateway for graceful degradation, we need to decouple API availability from the auth service's performance. This can be achieved by introducing asynchronous authentication and implementing circuit breakers.

The key idea is to design the gateway such that it can handle degraded states without bringing down the entire system. By using a stateless architecture and leveraging modern techniques like retries, exponential backoff, and fallback mechanisms, we can ensure that our APIs remain available even when the auth service encounters issues.

**Comparison:**

```
Operation	Outcome
Auth Success	Proceed to API logic
Auth Failure	Return unauthorized
```

**Code:**

```python
from fastapi import FastAPI, Depends
import asyncio

app = FastAPI()

# Simulate an external async auth service with potential delays or failures
async def fetch_auth_status(user_id):
    # Assume this function might throw exceptions due to network latency or failures
    await asyncio.sleep(2)  # Simulating a delay
    return True  # Example successful authentication

def authenticate_user(user_id: int):
    try:
        status = asyncio.run(fetch_auth_status(user_id))
        if not status:
            raise Exception("Auth failed")
    except Exception as e:
        print(f"Authentication failure: {e}")
        return False
    return True

@app.get("/protected_endpoint")
async def protected_endpoint(user_id: int):
    is_authenticated = authenticate_user(user_id)
    if is_authenticated:
        # Proceed with the actual API logic here
        return {"message": "Access granted"}
    else:
        # Fallback mechanism for unauthorized access
        return {"message": "Access denied"}

# Example of a circuit breaker pattern using `circuitbreaker` library in Python
from circuitbreaker import circuit_breaker

@authenticate_user.circuit_breaker(failmax=5, pause_for=10)  # Circuit breaker setup
def authenticate_user_with_circuit(user_id: int):
    return asyncio.run(fetch_auth_status(user_id))
```

**Example input / output:**

```text
# Example of input-output for the `protected_endpoint` function
user_id = 12345

# Normal scenario with successful authentication
response = protected_endpoint(user_id)
print(response)  # {"message": "Access granted"}

# Scenario with failed authentication (auth service down or slow)
response = protected_endpoint(67890)
print(response)  # {"message": "Access denied"}
```

**Data:**

```json
{
  "user_id": 12345,
  "is_authenticated": true
}
{
  "user_id": 67890,
  "is_authenticated": false
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Authenticated?
5. Proceed with API Logic
6. Fallback Mechanism ->Unauthorized Response
7. Circuit Breaker Triggers

