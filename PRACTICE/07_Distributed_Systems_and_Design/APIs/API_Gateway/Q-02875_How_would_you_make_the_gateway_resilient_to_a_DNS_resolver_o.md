---
type: "practice"
difficulty: "Deep"
---

# How would you make the gateway resilient to a DNS resolver outage when it resolves upstream hostnames dynamically; what caching, negative-TTL handling, and stale-on-error behavior do you configure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure resilience in an API Gateway setup where we resolve upstream hostnames dynamically, we need to implement robust caching mechanisms that handle DNS resolver outages effectively. This involves configuring caching with a negative TTL (Time To Live) and setting up stale-on-error behavior. Here's how you can approach this:

Understanding the Problem: When an API Gateway uses dynamic upstream hostnames, it often relies on DNS resolvers to resolve these names into IP addresses. A DNS resolver outage could cause delays or failures in resolving hostnames, impacting the gateway’s performance and availability.

Caching Configuration: We need to configure caching at multiple levels: client-side caching (in users’ browsers or proxies), API Gateway internal caching, and upstream service caching if applicable.

Negative TTL Handling: A negative TTL allows the cache to return a stale response when the actual resource is not available, but it can still serve this response until a fresh one becomes available. This helps in reducing the impact of DNS resolver outages on the gateway’s performance.

Stale-On-Error Behavior: Stale-on-error ensures that even if an upstream service returns an error, the cached response can be used for some time before it expires or gets refreshed. This is crucial because during a DNS outage, the actual state of the services might not reflect in the immediate errors.

#### Implementation
Steps:

Client-Side Caching: Configure your clients to cache responses appropriately so that they do not always hit the API Gateway for every request.
API Gateway Caching: Set up caching at the gateway level with a negative TTL (e.g., 10 seconds). This means if DNS resolution fails, the gateway will still serve the cached response until it can successfully resolve the hostname again.
Error Handling: Implement error handling logic to ensure that even if an upstream service is down or returns an error due to a DNS outage, the API Gateway can still return a cached and possibly stale response. This helps in maintaining availability during outages.

Monitoring and Logging: Regularly monitor the DNS resolver health and cache eviction policies. Log any issues to quickly identify and resolve them before they impact users significantly.

Fallback Mechanisms: Consider implementing fallback mechanisms where, if the primary service is down, a secondary or backup service can handle requests temporarily until the primary service recovers.

**Comparison:**

| Setting          | Value            | Explanation                                       |
|------------------|------------------|---------------------------------------------------|
| Negative TTL     | 10s              | Serve stale response for up to 10 seconds         |
| Stale On Error   | true             | Use cached response even if upstream returns error|
| Client Cache     | public, max-age=3600 | Allow clients to cache responses                  |

**Code:**

```python
# Example of setting up negative TTL in an API Gateway using an HTTP library like Flask
from flask import make_response

def cached_response(fn):
    def wrapper(*args, **kwargs):
        # Simulate DNS resolution and cache behavior
        try:
            hostname = resolve_hostname()
            response = fn(hostname)
        except DNSResolverOutage:
            # Return a stale response with negative TTL
            response = make_response("Fallback Service Response", 200)
            response.headers['Cache-Control'] = 'public, max-age=-10'
        return response

    return wrapper
```

**Example input / output:**

```text
Input (API Request):
GET /api/data HTTP/1.1
Host: example.com

Response (Initial):
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: public, max-age=-10

{"data": "Dynamic Data"}

Response (After DNS Outage but cached response still valid):
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: public, max-age=60

{"data": "Fallback Service Response"}
```

**Data:**

```json
{
  "cacheSettings": {
    "negativeTTL": "10s",
    "staleOnNetworkError": true,
    "clientCacheControl": "public, max-age=3600"
  },
  "resolverHealthCheck": {
    "interval": "5m",
    "timeout": "2s"
  }
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Resolve Hostname
5. DNS Resolved?
6. Get Data from UpstreamService
7. Use Cached Response(Stale-On-Error)
8. Return Data to Client

