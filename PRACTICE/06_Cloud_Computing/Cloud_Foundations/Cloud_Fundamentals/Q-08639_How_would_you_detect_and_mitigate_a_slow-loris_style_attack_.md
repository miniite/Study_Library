---
type: "practice"
difficulty: "Deep"
---

# How would you detect and mitigate a slow-loris style attack against your API tier that keeps connections open without sending data?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle a slow-loris attack, we need to understand its nature first. This type of DDoS attack keeps multiple HTTP connections open by sending partial requests and never completing them. The attacker abuses this behavior to tie up server resources, making legitimate traffic suffer.

We're looking for strategies that can both detect the attack in progress and mitigate its impact on our system. To do this from first principles, we'll consider several layers: network infrastructure, application layer, and database interactions.

Firstly, let's triage the problem by understanding what exactly needs to be detected:

Connection Behavior: We need to monitor how many connections are being opened but never completed.
Request Frequency: Detect if a significant number of partial requests are being sent over time.
Resource Consumption: Track the resources consumed per connection and overall.

Now, let's dive into a step-by-step solution:

Monitoring Connection Behavior

Implement connection pooling at the network level using tools like Nginx or HAProxy to monitor open connections.
Set up thresholds for maximum concurrent connections allowed from a single IP address.
Use metrics like total_requests and keep_alive_connections to identify potential slow-loris activity.

Rate Limiting

Introduce rate limiting at the application level using libraries such as Redis or Limiter middleware in frameworks like Express.js.
Define reasonable rates for API endpoints, e.g., 10 requests per minute from a single IP address.

Connection Timeout and Graceful Termination

Configure timeouts on HTTP connections to ensure they are closed after a certain period of inactivity.
Implement graceful termination logic that closes the connection if it detects no data has been sent for an extended period.

Behavioral Analysis

Use machine learning models trained on historical API usage patterns to detect anomalies.
Monitor client behavior, looking for signs of slow-loris activity such as repeated partial requests or prolonged inactivity.

Circuit Breaker Pattern

Introduce circuit breaker mechanisms at the API layer using libraries like Hystrix or Resilience4j.
This pattern can help degrade non-critical functionality when under attack, reducing load on the system.

Load Balancing and Distribution

Distribute traffic across multiple instances to ensure no single instance becomes a bottleneck.
Use load balancers that can dynamically adjust based on real-time performance metrics.

Database Optimization

Optimize database queries and indexing to reduce response time for each request.
Implement caching mechanisms at the API layer to serve frequently accessed data more quickly.

#### Closing Summary
To effectively mitigate a slow-loris attack, we need a multi-layered approach involving network monitoring, rate limiting, connection management, behavioral analysis, circuit breakers, load balancing, and database optimization. By implementing these strategies, we can both detect and handle the attack before it significantly impacts system performance.

#### Common Mistakes/Follow-Ups

Overlooking the importance of rate limiting or assuming that just using a firewall will suffice.
Not setting up proper monitoring and alerting mechanisms to quickly identify and respond to attacks.
Failing to optimize database queries, which can lead to unnecessary load on the server during an attack.

**Comparison:**

```
Metric	Description
total_requests	Total number of requests received from a specific IP address.
keep_alive_connections	Number of connections that are kept open without completing the request.
```

**Code:**

```python
# Example Rate Limiting with Redis
from redis import StrictRedis

redis_client = StrictRedis(host='localhost', port=6379, db=0)

def rate_limiter(ip_address):
    key = f"rate_limit_{ip_address}"
    if not redis_client.exists(key):
        # Increment the counter and set an expiration time (e.g., 1 minute)
        redis_client.setex(key, 60, value='1')
        return True
    else:
        current_value = int(redis_client.get(key))
        if current_value < 10:  # Allow up to 10 requests per minute
            redis_client.incrby(key, 1)
            return True
        else:
            # Exceeded rate limit, reject the request
            return False
```

**Example input / output:**

```text
Input:
- IP address: 192.168.1.100
- HTTP request (partial): GET /api/slowloris

Output:
- If rate limit has not been exceeded: "Request processed."
- If rate limit is exceeded: "Rate limit exceeded. Please try again later."

Dry-run example:
Input: IP 192.168.1.100, HTTP Request
Response: Rate limit exceeded (after sending 10 requests in the last minute)
```

**Data:**

```json
{
  "rate_limit_key": {
    "type": "string",
    "description": "Key used to track rate limiting for a specific IP address."
  },
  "max_requests_per_minute": {
    "value": 10,
    "description": "Maximum number of requests allowed per minute from the same IP address."
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive HTTP Request
4. Is Rate Limit Exceeded?
5. Reject Request
6. Process Request
7. Log Violation
8. Monitor Connection Behavior

