---
type: "practice"
difficulty: "Deep"
---

# Your service's REST calls to a dependency use a shared connection pool, and one endpoint's slowness starves all others. How do you implement bulkheads so each downstream gets isolated resources?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, implementing bulkheads is crucial to ensure that slow endpoints don't overwhelm or starve out other dependent services. A bulkhead, in essence, acts as a mechanism that isolates parts of your system from each other, thereby preventing the failure of one part from cascading into others.

This question is essentially about understanding how to design and implement a resilient system architecture using circuit breakers and rate limiting—both of which are key components of bulkheads. The interviewer is probing for knowledge on service isolation, fault tolerance, and the trade-offs between performance and reliability.

#### Assumptions

We are working with a REST API that uses a connection pool to interact with a downstream dependency.
Slowness in one endpoint is causing latency or resource starvation issues for other endpoints.
The solution needs to be scalable and robust enough to handle varying loads and potential failures.

Step-by-step solution:

Circuit Breaker Implementation:

A circuit breaker acts as a switch that, when triggered by slow or failing requests, will stop further calls from being sent downstream and return an error response.
Use a library like Hystrix (Java) or Resilience4j (multi-language) to implement the circuit breaker pattern.

Rate Limiting:

Implement rate limiting on each endpoint to prevent any single endpoint from overwhelming the shared connection pool.
Use libraries such as Guava's RateLimiter, AWS API Gateway throttling settings, or Envoy for more advanced rate limiting capabilities.

Connection Pool Management:

Ensure that the connection pool is properly configured to handle both maximum connections and timeout settings.

#### Cons
ider implementing connection pooling strategies like "steal" or "spill," where extra idle connections from one service can be used by another if necessary but with careful monitoring to avoid overloading.

Service Discovery and Load Balancing:

Use a load balancer that supports health checks and has built-in mechanisms for handling slow endpoints.
Implement a strategy like "failover" or "backup" services, where if an endpoint is slow, requests can be routed to alternative instances.

Monitoring and Metrics:

Set up comprehensive monitoring and metrics on the downstream dependency calls.
Use tools like Prometheus and Grafana for real-time insights into service health and performance.

#### Closing Summary
To address this issue effectively, we need to implement a combination of circuit breakers and rate limiting strategies. Circuit breakers help prevent cascading failures by isolating problematic services, while rate limiting ensures that no single endpoint overloads the system. Additionally, optimizing connection pooling and leveraging advanced load balancing can further enhance resilience.

#### Common Mistakes/Follow-ups

Overlooking the importance of monitoring and metrics.
Failing to properly configure circuit breakers, which could result in not capturing slow or failing service states effectively.
Not considering the impact on other services when implementing rate limiting policies.

**Comparison:**

```
Endpoint	Circuit Breaker Thresholds	Rate Limiting Settings
A	50% failure ratio / 100 reqs	500 reqs/second
B	70% failure ratio / 200 reqs	300 reqs/second
```

**Code:**

```java
// Example Hystrix implementation
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

public class DownstreamServiceCall extends HystrixCommand<Void> {
    public DownstreamServiceCall() {
        super(HystrixCommandGroupKey.Factory.asKey("DownstreamService"));
    }

    @Override
    protected Void run() throws Exception {
        // Make the REST API call here and handle any exceptions
        return null;
    }
}
```

**Example input / output:**

```text
Input:
- Slow endpoint request rate is consistently above the threshold.
- Connection pool exhaustion occurs frequently.

Output:
- Circuit breaker trips for slow endpoints, preventing further calls from being made and returning immediate errors to clients.
- Rate limiting enforces a maximum number of requests per second, ensuring fair usage of the connection pool.
```

**Data:**

```json
{
  "circuitBreakerSettings": {
    "failureRatioThreshold": 0.5,
    "requestVolumeThreshold": 100,
    "waitDurationInOpenState": 30000
  },
  "rateLimitingSettings": {
    "limitForPeriod": 500,
    "periodInMilliseconds": 1000
  }
}
```

**Flow:**

1. Client Request
2. Circuit Breaker Decision
3. Rate Limiter Check
4. Make API Call (if conditionsmet)
5. Handle Response or Error

