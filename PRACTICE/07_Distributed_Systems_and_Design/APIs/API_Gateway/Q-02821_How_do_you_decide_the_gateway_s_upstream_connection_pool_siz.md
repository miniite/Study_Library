---
type: "practice"
difficulty: "Deep"
---

# How do you decide the gateway's upstream connection pool size per service given a fixed number of backend pods, and what symptom tells you the pool is too small (queuing) vs too large (backend overload)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with determining an optimal upstream connection pool size per service in an API Gateway, given a fixed number of backend pods. The key goal is to balance between queuing and backend overload, ensuring efficient use of resources while maintaining high availability.

The gateway's connection pool size dictates how many concurrent requests can be sent to the backend services at any one time. A correctly sized pool will ensure that there are enough connections to handle incoming traffic without overwhelming the backend or causing excessive queuing in the API Gateway. The symptom for an incorrectly sized pool is either queuing (backed up requests) or overload (backend services under too much pressure).

#### Triaging the Question

The question revolves around optimizing resource utilization and ensuring that the gateway can efficiently manage incoming requests while minimizing both queueing delays and backend service stress.

#### Assumptions
We have a fixed number of backend pods.
The API Gateway is configured to distribute traffic evenly among these pods.
Each request from the client eventually needs to reach one specific pod in the backend cluster.

#### First Principles Solution

Determine Request Rate: Measure or estimate the average incoming request rate (R) per second that your gateway can expect.

Identify Backend Capacity: Determine the maximum number of concurrent connections each backend pod can handle without overloading. This can be empirically determined by testing and monitoring.

Calculate Pool Size: Use these numbers to calculate an optimal pool size for each service.

Let ( N ) be the total number of backend pods.
Let ( C ) be the maximum concurrent connections per pod before overload (measured or estimated).
The upstream connection pool size (( P )) for each service can be calculated as: [ P = \left\lceil \frac{R}{N} \right\rceil \times C ] Here, ( \left\lceil x \right\rceil ) denotes the ceiling function, ensuring we round up to the nearest integer.

Monitor and Adjust: Continuously monitor the gateway's behavior:

Queuing: If requests start queuing in the gateway (indicating it can't forward all incoming requests), increase the pool size.
Overload: If backend services show signs of stress (e.g., high CPU utilization, response time increases), decrease the pool size.

#### Closing Summary
and Common Mistakes
Symptoms: Queuing is indicated by increased latency in request processing. Backend overload is shown through higher CPU and memory usage on the pods.
Mistakes to Avoid:
Failing to account for variability in request rates or pod capacity.
Setting pool sizes too low, leading to frequent queuing and poor user experience.
Over-provisioning, which wastes resources and may introduce unnecessary complexity.

#### Example
Calculation

Suppose we have an API Gateway receiving 100 requests per second (R = 100) with 5 backend pods (( N )), each capable of handling up to 20 concurrent connections before overload (( C = 20 )): [ P = \left\lceil \frac{100}{5} \right\rceil \times 20 = 20 \times 20 = 400 ] So, each service should have a pool size of 400 connections.

Summary and Follow-Ups

Understanding the relationship between request rate, backend capacity, and connection pool size is crucial for optimizing API Gateway performance. Regular monitoring and adjustment based on observed behavior are key to maintaining an efficient setup.

Common follow-ups could involve discussing load balancing strategies, retry policies, or how to handle dynamic changes in traffic patterns.

**Comparison:**

```
Parameter	Value
Request Rate	100 requests/s
Backend Pods	5
Max Concurrent	20
Connection Size	400
Gateway Behavior	Symptoms
Queuing	Increased latency
Overload	High CPU/Memory use
```

**Code:**

```python
def calculate_pool_size(request_rate, num_pods, max_concurrent_connections_per_pod):
    pool_size = (request_rate / num_pods) * max_concurrent_connections_per_pod
    return int(pool_size)
```

**Example input / output:**

```text
Input:
- Request Rate (R): 100 requests/s
- Number of Backend Pods (N): 5
- Max Concurrent Connections Per Pod (C): 20

Output:
- Upstream Pool Size (P) per Service: 400 connections
```

**Data:**

```yaml
# Example configuration for backend pods and API Gateway settings
api_gateway_config:
  request_rate: 100 # requests per second
  num_backend_pods: 5
  max_concurrent_connections_per_pod: 20

backend_pod_capacity:
  max_concurrent_connections: 20 # per pod, before overload
```

**Flow:**

1. No
2. Yes
3. Queuing
4. No
5. Measure Request Rate
6. Identify Backend Capacity
7. Is Pool Size Optimal?
8. Set Larger Pool Size
9. Monitor Gateway Behavior
10. Detect Queuing or Overload?
11. Set Larger Pool Size
12. Gateway Operating Normally

