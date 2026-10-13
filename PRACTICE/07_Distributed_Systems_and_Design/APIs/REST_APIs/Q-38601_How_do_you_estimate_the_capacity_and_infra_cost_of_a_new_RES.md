---
type: "practice"
difficulty: "Deep"
---

# How do you estimate the capacity and infra cost of a new REST endpoint before launch: expected QPS, payload size, cache hit ratio, and downstream calls? Walk me through a back-of-envelope for a product feed.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To estimate the capacity and infrastructure costs for a new REST API endpoint like a product feed, we need to consider several key factors including expected QPS (Queries Per Second), payload size, cache hit ratio, and downstream calls. This back-of-the-envelope calculation will give us a high-level understanding of both the performance requirements and cost implications.

Firstly, let's break down what these terms mean:

Expected QPS: The number of requests per second that the API needs to handle.
Payload Size: The size of the data being transmitted in each request/response cycle.
Cache Hit Ratio: The percentage of requests that can be served directly from cache without hitting the database or other backends.
Downstream Calls: External services or APIs that the product feed might call, which also add to latency and cost.

To start our estimation, we need some initial assumptions:

Assume an average payload size of 10KB per request/response cycle.
Assume a cache hit ratio of 75% (meaning only 25% of requests will require hitting downstream services or databases).
Assume that the API will receive around 100 QPS initially, with a potential growth rate of +30% annually.
Step-by-Step Calculation
1. QPS and Throughput

Assume an average response time of 20ms (this includes network latency). To handle 100 QPS, we need to ensure that the system can process at least 100 requests per second with minimal delay.

[ \text{Throughput} = \frac{\text{QPS}}{\text{Response Time in Seconds}} = \frac{100}{\frac{20 \times 10^{-3}}{1}} = 5000 , \text{requests per second} ]

This means we need a system capable of handling 5000 requests per second at peak load.

2. Payload Size and Bandwidth

With an average payload size of 10KB, the bandwidth required for 100 QPS is:

[ \text{Bandwidth} = \text{QPS} \times \text{Payload Size} = 100 \times 10 \times 1024 , \text{bytes/second} = 1.024 , \text{MB/s} ]

3. Cache Hit Ratio and Downstream Calls

If the cache hit ratio is 75%, then 25% of requests (i.e., 25 QPS) will require hitting downstream services. Assuming each of these calls takes an additional 100ms, we add this to our initial processing time:

[ \text{Additional Latency} = 25 \times 100 \times 10^{-3} , \text{seconds/second} = 2.5 , \text{seconds/second} ]

Thus, the total latency including cache misses would be approximately (20 + 2.5 = 22.5) ms.

4. Cost Estimation

For cost estimation, we need to consider both hardware and operational costs:

Hardware Costs: Cloud providers like AWS or GCP offer scalable instances. For a throughput of 5000 requests per second, an M6g.medium might be sufficient.
Operational Costs: Maintenance, monitoring tools, and other cloud services can add around $10-$20 per month.

#### Closing Summary

In summary, to handle 100 QPS with an average payload size of 10KB and a cache hit ratio of 75%, we need hardware capable of processing at least 5000 requests per second. The bandwidth required is approximately 1 MB/s. Additional latency for downstream calls needs to be considered, increasing the total response time by about 2.5 ms.

#### Common Mistakes/Follow-ups
Underestimating the initial QPS and cache hit ratio.
Not considering the additional latency from downstream calls.
Overlooking operational costs in the total cost of ownership.
Failing to scale appropriately for growth (e.g., only estimating for current load).

These calculations provide a robust foundation, but real-world scenarios may require more detailed analysis and iterative testing.

**Comparison:**

```
Metric	Value
QPS	100
Payload Size	10KB
Cache Hit Ratio	75%
Response Time	20ms
Additional Latency	2.5 ms
```

**Code:**

```python
# Example code snippet to handle QPS calculation
def estimate_qps_throughput(qps, response_time_ms):
    throughput = qps / (response_time_ms * 1e-3)
    return throughput

estimated_throughput = estimate_qps_throughput(100, 20)
print(f"Estimated Throughput: {estimated_throughput} requests per second")
```

**Example input / output:**

```text
Input:
- QPS = 100, Payload Size = 10KB, Cache Hit Ratio = 75%, Response Time = 20ms

Output:
- Throughput = 5000 requests per second
- Bandwidth = 1.024 MB/s
- Additional Latency = 2.5 ms (including cache misses)
```

**Data:**

```json
{
  "payload_size": 10 * 1024,       // 10KB in bytes
  "cache_hit_ratio": 0.75,
  "initial_qps": 100,
  "additional_latency_ms": 100    // Additional latency for cache misses
}
```

**Flow:**

1. Initial QPS
2. Calculate Throughput
3. Estimate Bandwidth
4. Consider Downstream Calls
5. Adjust Response Time
6. Estimate Cost

