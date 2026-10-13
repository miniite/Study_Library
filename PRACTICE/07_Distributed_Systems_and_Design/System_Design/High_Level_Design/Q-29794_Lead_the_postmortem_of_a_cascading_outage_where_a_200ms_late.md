---
type: "practice"
difficulty: "Expert"
---

# Lead the postmortem of a cascading outage where a 200ms latency bump in one service exhausted thread pools across a dozen callers — what systemic (not local) fixes do you mandate org-wide afterward?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a recent postmortem, we identified a cascading outage due to a 200ms latency bump in a critical service, which led to thread pool exhaustion and subsequent failures across multiple downstream services. The systemic issue was the lack of proper handling of latency variability and concurrency limits. To prevent such incidents from recurring, we must mandate several organizational changes focusing on improved resilience, monitoring, and capacity planning.

Firstly, it’s crucial to recognize that any service in our system should be designed with built-in buffers for unexpected latency spikes. This means implementing mechanisms like circuit breakers, rate limiting, and exponential backoff to handle such situations without overwhelming the system's resources. Additionally, we need to ensure that our thread pools are not only properly sized but also dynamically adjusted based on real-time load conditions.

**Comparison:**

```
Metric	Current Value	Target Value
Thread Pool Size Adjustment Logic	Static	Dynamic based on real-time load
Circuit Breaker Usage	Not consistently implemented	Mandatory for all services
Exponential Backoff Mechanism	Rarely used	Standard practice in high-latency scenarios
```

**Code:**

```python
# Example of dynamic thread pool resizing in Python using concurrent.futures
import concurrent.futures

def resize_thread_pool(current_load):
    # Adjust the number of threads based on current load
    new_pool_size = max(1, min(20, current_load * 2))  # Simplified logic for illustration
    with concurrent.futures.ThreadPoolExecutor(max_workers=new_pool_size) as executor:
        results = list(executor.map(process_request, requests))
```

**Example input / output:**

```

Dry-run of adjusting thread pool size based on load:

PLAINTEXT
Copy
Current Load: [10 requests]
Dynamic Resizing: new_pool_size = max(1, min(20, 10 * 2)) => 20 threads

[Time passes and load increases to 20 requests]

Current Load: [20 requests]
Dynamic Resizing: new_pool_size = max(1, min(20, 20 * 2)) => 40 threads (scaled down if necessary)
```

**Data:**

```json
{
  "requests": [
    {"id": 1, "processing_time_ms": 50},
    {"id": 2, "processing_time_ms": 300}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Service Request
4. Circuit Breaker
5. Exponential Backoff
6. Thread Pool Request
7. Load Threshold Exceeded?
8. Rate Limiting
9. Process Request

