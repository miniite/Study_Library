---
type: "practice"
difficulty: "Deep"
---

# How would you architect an inference gateway that does auth and rate limiting per API key so one customer cannot exhaust your GPU capacity?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a robust inference gateway that handles authentication, rate limiting, and quota management based on API keys to prevent any single customer from exhausting your GPU capacity, we need to design a system with multiple layers of protection. The core idea is to use a combination of edge gateways, centralized auth servers, and rate-limiting policies enforced by both the edge and the backend services.

First, let's triage the question:

What is being asked? This involves designing an inference gateway that authenticates API keys, enforces rate limits based on those keys, and ensures that no single customer can overuse GPU resources.
Key assumptions: We assume a distributed system with edge gateways (e.g., AWS Lambda or GCP Cloud Functions) handling initial authentication and rate limiting. The backend services will have further protections to ensure resource fairness.

Now, let's dive into the solution from first principles:

Edge Gateway Layer:
Use an edge gateway to perform initial checks on API keys and enforce a basic set of rate limits.

#### Example
AWS API Gateway or GCP Cloud Endpoints can be used here.
Centralized Authentication Service:
Implement a centralized authentication service that handles token validation, quota enforcement, and log aggregation for audit purposes.
Use OAuth 2.0 with JWT tokens to securely pass credentials between the edge gateway and backend services.
Rate Limiting Policies:
Apply rate limits at both the edge and backend levels using configurable thresholds based on API key metadata.
Backend Service Layer:
Implement a scalable ML model serving system (e.g., TensorFlow Serving, Sagemaker) with per-API-key quota enforcement.
Monitoring & Logging:
Set up comprehensive logging and monitoring to detect abuse patterns or anomalies in resource usage.

Here’s a step-by-step breakdown of the solution:

Edge Gateway Design:

Yes
No
API Request
Is API Key Valid?
Apply Basic Rate Limits
Forward Request to Backend
Deny Request & Log

Centralized Authentication Service:

Yes
No
Authenticate API Key
Is Quota Exceeded?
Log Abuse & Deny Request
Grant Access & Send Token toEdge Gateway

Backend Service Layer:

Yes
No
Token Received
Is Quota Exceeded?
Deny Request & Log
Predict & Forward Result

Rate Limiting Enforcement:

Implement token-based rate limiting using tools like AWS WAF, or custom solutions leveraging Redis and Lua scripts for complex rules.

#### Example
redis.lua script to enforce rate limits.

Scalability & Fault Tolerance:

Use load balancers (e.g., AWS ALB) to distribute requests across multiple instances of the edge gateway and backend services.
Implement auto-scaling policies based on CPU utilization or custom metrics.

#### Closing Summary
The overall architecture includes an initial layer of rate limiting and authentication at the edge, followed by a centralized service for detailed quota management. Backend services enforce final checks to ensure fairness. Monitoring tools are critical for detecting abuse patterns early.

#### Common Mistakes
& Follow-Ups:

Mistake 1: Failing to implement rate limiting on both the edge gateway and backend levels can lead to resource exhaustion.

#### Follow-Up
1: Discuss how to handle dynamic quotas based on user behavior or time of day.
Mistake 2: Not using secure authentication mechanisms like OAuth 2.0 with JWT tokens can expose API keys to security risks.

#### Follow-Up
2: Explore other authentication methods and their implications, such as token revocation and reissuance.

**Comparison:**

| API Key | Rate Limit (req/min) | Current Count | Expiry Time         |
|---------|---------------------|---------------|--------------------|
| 12345   | 10                  | 9             | 60 seconds from now |

**Code:**

```python
# Example of a simple rate limiting function using Redis
import redis

def enforce_rate_limit(api_key):
    r = redis.Redis(host='localhost', port=6379, db=0)
    key = f"rate_limit:{api_key}"
    if not r.exists(key):  # Check if the key exists (first request)
        r.set(key, 1, ex=60)  # Set initial value with expiration
    else:
        count = int(r.get(key))
        if count >= 5:  # Example threshold of 5 requests per minute
            return False  # Rate limit exceeded
        r.incr(key)
    return True  # Allow request
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `GET /inference?key=1234567890` | 200 OK, request processed |

**Data:**

```json
{
  "api_key": "1234567890",
  "quota_exceeded": false,
  "requests_per_minute": 4,
  "remaining_limit": 1
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Is API Key Valid?
5. Apply Basic Rate Limits
6. Forward Request to Backend
7. Deny Request & Log

