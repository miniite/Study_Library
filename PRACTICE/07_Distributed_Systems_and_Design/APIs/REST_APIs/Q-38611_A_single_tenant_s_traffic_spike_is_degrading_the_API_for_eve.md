---
type: "practice"
difficulty: "Expert"
---

# A single tenant's traffic spike is degrading the API for everyone. How do you implement fair-use rate limiting and per-tenant quotas without a central bottleneck, and how do you communicate limits to clients?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To address the issue of a single tenant's traffic spike degrading the API for everyone, we need to implement fair-use rate limiting and per-tenant quotas in a distributed manner to avoid central bottlenecks. This involves leveraging Redis for real-time state management, configuring rate limits at both application and database levels, and communicating these limits effectively to clients.

#### Triage
The key aspects of the question revolve around ensuring that high traffic from one tenant does not affect others by implementing fair-use rate limiting and per-tenant quotas. The interviewer is probing your knowledge of distributed systems, state management, performance optimization, and client communication strategies. Assumptions include a multi-tenant architecture where each tenant has its own database or resource pool.

#### Solution

State Management with Redis: Use Redis as a cache and state manager to keep track of the quota usage for each tenant. This allows us to handle real-time rate limiting without relying on a central bottleneck.
Rate Limiting at Application Level: Implement rate limits within your application code, using middleware or decorators. For example, in Node.js, you could use express-rate-limit.
Database-Level Rate Limiting: Configure database-level rate limits to prevent excessive queries from overwhelming the backend services. This can be achieved through connection pooling and query throttling.
API Gateway for Centralization: Use an API gateway (like Kong or Traefik) that can enforce global rate limits before routing requests to individual tenants. The gateway should also act as a load balancer, distributing traffic evenly across multiple instances of the backend service.
Client Communication Strategy:
Send 429 Too Many Requests responses when clients exceed their allowed limits.
Provide clear and concise documentation on rate limits and quotas for each tenant.
Implement retry logic in client-side code to handle temporary rate limit issues.

#### Closing Summary
By implementing a distributed approach with Redis, configuring rate limits at multiple levels (application, database), and using an API gateway for centralized control, we can ensure that high traffic from one tenant does not degrade the performance of other tenants. Effective communication strategies are crucial to prevent client confusion and maintain service availability.

#### Common Mistakes/Follow-Ups

Not Accounting for Connection Pools: Failing to configure connection pools in databases can lead to performance issues.
Ignoring Client-Side Retry Logic: Not advising clients on how to handle rate limit errors can result in suboptimal user experiences.
Overlooking Centralized Monitoring: Lack of centralized monitoring and logging can make it difficult to diagnose and resolve issues.

**Comparison:**

| Tenant ID | Rate Limit Quota (reqs/min) | Remaining Usage | Time of Last Update |
|-----------|-----------------------------|-----------------|--------------------|
| 123       | 100                         | 95              | 2023-10-01T14:00:00Z |

**Code:**

```javascript
// Example using express-rate-limit for application-level rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100,            // limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

**Example input / output:**

```text
Input:
GET /api/resource

Output (when tenant quota is exceeded):
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{
  "message": "Rate limit exceeded",
  "retry-after": 60,
  "allowed-requests-per-minute": 100
}
```

**Data:**

```json
{
  "tenantId": "123",
  "rateLimitQuota": {
    "window": "60s",
    "limit": 100
  },
  "remainingUsage": 95,
  "timeOfLastUpdate": "2023-10-01T14:00:00Z"
}
```

**Flow:**

1. No
2. Yes
3. API Gateway
4. Rate Limit Middleware
5. Backend Service (Redis)
6. Client
7. API Gateway
8. Is Rate Limit Exceeded?
9. Proceed with Request
10. Send 429 Response,Retry-After Header

