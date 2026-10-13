---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd tune a client-side timeout for a downstream service given its p50 p99 and p999 latency. Why does a timeout set at p99 cause a retry storm during a latency shift?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tune a client-side timeout for a downstream service, we need to understand the distribution of its latencies and how they affect the overall system performance under varying conditions. Specifically, setting a timeout based on p50, p99, or p999 can have different implications for reliability and performance. Given that a timeout set at p99 causes a retry storm during a latency shift, we must explore why this happens and how to mitigate it.

Firstly, let's reframe the question: What are the latency percentiles (p50, p99, p999) and what do they represent? The p50 latency is the 50th percentile, meaning half of the requests take longer than this value. The p99 latency represents that 99% of the requests complete within this time frame, leaving just 1% with higher latencies. Similarly, p999 indicates that only 0.1% of requests exceed this threshold.

When setting a client-side timeout to match these percentiles, we need to ensure it aligns with expected service performance but also provides enough buffer for rare events. Setting the timeout at p99 means it will fail half of the p99 latency requests, which are still within the "normal" operation range. However, when there's a sudden shift in latency due to factors like network congestion or upstream issues, these p99 requests can suddenly become p95 or worse, triggering retries.

To understand why this leads to a retry storm:

Latency Shift: When service latencies shift suddenly, the number of failed requests increases significantly.
Retry Mechanism: Retries are designed to ensure resiliency but when many requests fail at once due to a latency spike, the system can become overwhelmed with retries, leading to further delays and potential cascading failures.

To mitigate this issue, we need to consider multiple strategies:

Increase Timeout Gracefully: Instead of setting it exactly at p99 or even p95, increase it by a buffer that accounts for known variability.
Dynamic Adjustment: Implement dynamic timeout adjustment based on recent service health metrics.
Rate Limiting and Circuit Breakers: Use rate limiting to prevent overwhelming the system during high latency events, and circuit breakers to isolate failing services.

Let's go through this step-by-step:

Assess Current Latency Distribution:
Collect historical latency data for p50, p99, and p999.
Use this data to set initial timeout values that provide a buffer against known variability.
Implement Dynamic Timeout Adjustment:
Monitor real-time service health indicators (e.g., throughput, error rates).
Adjust the timeout dynamically based on these metrics.
Rate Limiting and Circuit Breakers:
Integrate rate limiting to prevent too many retries in a short period.
Configure circuit breakers to handle unexpected failures gracefully.

In summary, setting a timeout at p99 introduces a significant risk of retry storms during latency shifts because it fails more requests than necessary. By increasing the buffer, dynamically adjusting timeouts, and implementing robust resiliency mechanisms, we can mitigate these issues and improve system reliability.

**Comparison:**

```
Latency Percentile	Description
p50	Median latency
p99	99th percentile
p999	99.9th percentile
```

**Code:**

```python
# Example of dynamic timeout adjustment in Python
import time

def dynamic_timeout(latencies):
    p50 = latencies['p50']
    p99 = latencies['p99']
    buffer = 1.2  # Adjust as needed based on historical data
    return max(p99 * buffer, p50 + (p99 - p50))

# Simulated latency data
latencies = {'p50': 100, 'p99': 300}
timeout = dynamic_timeout(latencies)
print(f"Dynamic Timeout: {timeout}")
```

**Example input / output:**

| Input (Latency Percentiles) | Output (Timeout Value) |
|-----------------------------|------------------------|
| p50: 100, p99: 300, p999: 600 | Timeout: 360           |

**Data:**

```json
{
  "p50": 100,
  "p99": 300,
  "p999": 600
}
```

**Flow:**

1. Yes
2. No
3. Monitor Latency Distribution
4. Determine Initial Buffer
5. Is Latency Stable?
6. Use Static Timeout Value
7. Adjust Timeout Dynamically
8. Implement Rate Limiting andCircuit Breakers
9. Handle Unexpected FailuresGracefully

