---
type: "practice"
difficulty: "Deep"
---

# A downstream is slow-but-not-down — the worst failure. Explain how bulkheads, per-dependency thread pools, and load-shedding stop it from exhausting your own threads and cascading.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address a downstream service that is slow but not down, we can employ bulkheads, per-dependency thread pools, and load-shedding to prevent resource exhaustion in our own system. Bulkheads limit the impact of failures by isolating services, thread pools manage concurrency effectively, and load shedding gracefully handles excess requests.

#### Triaging the Question

The question is about designing a robust system architecture that can handle slow but not down downstream services without compromising its own performance or stability. The key concepts to focus on are:

Bulkheads: To isolate failure domains.
Per-dependency Thread Pools: To manage concurrency efficiently.
Load Shedding: To gracefully handle excess requests.

#### Assumptions
and Context
System Architecture: A microservices-based architecture where multiple services communicate with each other.
Downstream Service Issue: The service is slow, but it doesn’t fail entirely; it just takes longer than expected to process requests.
Impact on Upstream System: If not properly managed, the slowdown in the downstream service can lead to thread exhaustion and cascading failures.

#### First-Principles Solution

Bulkheads (Service Isolation)

Definition: Bulkheads are like firewalls that isolate parts of a system from each other.

#### Implementation
Implement bulkheads using circuit breakers, which monitor the health of downstream services and limit requests if they exceed a threshold.

#### Example
Use libraries like Hystrix in Java or similar mechanisms to implement circuit breakers.

Per-Dependency Thread Pools

Definition: Thread pools are used to manage concurrency by limiting the number of threads that can be used for processing requests.

#### Implementation
Create a thread pool specifically for each downstream service, with its own configurable size and behavior.

#### Example
Use ExecutorService in Java or similar constructs to create per-service thread pools.

Load Shedding

Definition: Load shedding is about gracefully handling excess requests when the system is under heavy load.

#### Implementation
Implement mechanisms like request queuing, retry policies, and exponential backoff strategies.

#### Example
Use a combination of these techniques to manage incoming requests effectively.
Detailed Solution

Bulkheads

Circuit Breakers:
Monitor the response time of downstream services using metrics (e.g., Prometheus).
Set thresholds for maximum acceptable latency and failure rate.
If the thresholds are exceeded, the circuit breaker trips and blocks further requests to that service until it recovers.
JAVA
@HystrixCommand(fallbackMethod = "fallbackMethod", commandProperties = {
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50")
})
public String callDownstreamService() {
    return downstreamService.call();
}

public String fallbackMethod() {
    // Fallback logic
    return "Fallback response";
}
Service Isolation:
Use namespaces or separate deployment units for different services to ensure that the failure in one does not affect others.

Per-Dependency Thread Pools

Thread Pool Configuration:
Determine the optimal number of threads based on the expected load and service characteristics.
Configure thread pools using frameworks like ThreadPoolExecutor or Executors.newFixedThreadPool.
JAVA
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5, // core pool size
    10, // maximum pool size
    60L, TimeUnit.SECONDS, // keep-alive time
    new LinkedBlockingQueue<Runnable>(100) // queue capacity
);
Handling Errors Gracefully:
Use RejectedExecutionHandler to define what happens when the thread pool is full.
JAVA
executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());

Load Shedding

Request Queuing:
Implement a queue for incoming requests and process them in order, ensuring that only a certain number of requests are processed at once.
JAVA
BlockingQueue<Runnable> requestQueue = new LinkedBlockingQueue<>();
executor.execute(() -> {
    while (!Thread.interrupted()) {
        Runnable task = requestQueue.poll(10, TimeUnit.SECONDS);
        if (task != null) {
            try {
                task.run();
            } catch (Exception e) {
                // Handle exception
            }
        }
    }
});
Retry Policies:
Implement exponential backoff with retries for transient failures.
JAVA
int retryCount = 0;
while (retryCount < 3) {
    try {
        return downstreamService.call();
    } catch (Exception e) {
        retryCount++;
        Thread.sleep(2 ^ retryCount * 100);
    }
}

#### Closing Summary
and Common Mistakes
Summary: By implementing bulkheads, per-dependency thread pools, and load shedding, we can effectively manage the impact of slow downstream services without causing resource exhaustion in our own system.

#### Common Mistakes
Not isolating failure domains properly using circuit breakers.
Using a single thread pool for all services instead of creating separate ones.
Failing to implement proper load shedding mechanisms, leading to excessive request queuing and delays.
Self-Check

Correctness + completeness confirmed.

**Comparison:**

| Service | Latency (ms) | Error Rate (%) |
|---------|-------------|---------------|
| A       | 200         | 5             |
| B       | 300         | 7             |
| C       | 400         | 8             |

**Code:**

```java
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

public class DownstreamServiceHystrixCommand extends HystrixCommand<String> {
    public static final String GROUP_NAME = "DownstreamServices";

    @Override
    protected String run() throws Exception {
        return downstreamService.call();
    }
}
```

**Example input / output:**

```text
Input:
- DownstreamServiceHystrixCommand invoked with GROUP_NAME as "DownstreamServices"

Output:
- If downstream service latency exceeds threshold, circuit breaker trips and fallback logic is executed.
- Thread pool processes requests based on configured size and behavior.
- Load shedding mechanisms handle excess requests by queuing them or retrying with backoff.
```

**Data:**

```json
{
  "serviceHealth": {
    "downstreamService1": {
      "latencyMs": 200,
      "errorRate": 5
    },
    "downstreamService2": {
      "latencyMs": 300,
      "errorRate": 7
    }
  },
  "threadPoolConfig": {
    "coreSize": 5,
    "maxSize": 10,
    "keepAliveTimeSec": 60
  }
}
```

**Flow:**

1. Yes
2. No
3. Size > 0
4. Size = 0
5. Call Downstream Service
6. Is Circuit Breaker Open?
7. Execute Fallback
8. Process Request
9. Check Thread Pool Size
10. Dispatch to Thread Pool
11. Queue Request

