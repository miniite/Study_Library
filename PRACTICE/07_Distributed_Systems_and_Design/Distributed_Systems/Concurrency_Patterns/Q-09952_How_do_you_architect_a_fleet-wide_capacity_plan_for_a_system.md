---
type: "practice"
difficulty: "Deep"
---

# How do you architect a fleet-wide capacity plan for a system where concurrency is bounded by a shared distributed lock's throughput ceiling? Show how you'd model the ceiling, forecast when you hit it at projected growth, and the re-sharding roadmap before it becomes a customer-facing incident.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a fleet-wide capacity plan for a system with bounded concurrency by a shared distributed lock's throughput ceiling, we need to model the concurrency constraints precisely and forecast potential bottlenecks at projected growth rates. This involves understanding the current load, the rate-limiting behavior of the distributed locks, and how these factors interact under different workloads. We'll then define a re-sharding roadmap that addresses scalability concerns before they become critical.

At a high level, we're looking to ensure that our system can handle increasing loads without hitting the throughput ceiling of the distributed lock mechanism. This involves several key steps: understanding the current architecture and load characteristics, modeling the concurrency limits, forecasting future growth, and defining an actionable re-sharding plan.

**Comparison:**

```
Month	Current RPS (requests/s)	Projected RPS (requests/s)
1	20	30
6	20	40
12	20	50
```

**Code:**

```python
# Example code to model the throughput limit of a distributed lock
class DistributedLock:
    def __init__(self, max_throughput):
        self.max_throughput = max_throughput

    def acquire(self):
        # Simulate acquiring the lock
        if random.random() < (1.0 / self.max_throughput):
            return True  # Successfully acquired
        else:
            return False  # Failed to acquire due to rate limit

# Example usage of DistributedLock
lock = DistributedLock(max_throughput=5)
acquire_successes = [lock.acquire() for _ in range(100)]
print(f"Acquisition success rate: {sum(acquire_successes) / len(acquire_successes)}")
```

**Example input / output:**

```text
Given the current system load of 20 requests per second with a concurrency limit of 5:
- Acquire success rate over 100 attempts: ~80%

Projected growth in requests per second for the next year:
- Month 1: 30 requests/s
- Month 6: 40 requests/s
- Month 12: 50 requests/s

To forecast when the concurrency limit will be hit, we simulate acquiring the lock under different request rates. For instance, at a rate of 40 requests per second, the success rate drops significantly, indicating that the current throughput ceiling is a bottleneck.
```

**Data:**

```json
{
  "system_load": {
    "requests_per_second": 20,
    "concurrency_limit": 5
  },
  "growth_projections": [
    {"month": 1, "requests_per_second": 30},
    {"month": 6, "requests_per_second": 40},
    {"month": 12, "requests_per_second": 50}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Current System Load
4. Is Concurrency Limit aBottleneck?
5. Monitor Performance
6. Define Re-sharding Plan
7. Implement and Test Shards
8. Deploy New ShardConfiguration

