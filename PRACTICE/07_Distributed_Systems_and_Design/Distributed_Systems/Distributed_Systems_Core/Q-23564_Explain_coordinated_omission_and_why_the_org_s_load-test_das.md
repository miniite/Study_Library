---
type: "practice"
difficulty: "Deep"
---

# Explain coordinated omission and why the org's load-test dashboards keep lying about tail latency while production burns. Redesign both the load generator and the production measurement so p99.9 reflects what users actually experience.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Coordinated Omission is a critical issue in distributed systems, particularly when dealing with highly concurrent workloads. In simple terms, coordinated omission occurs when multiple components in a system fail to process certain messages or events due to race conditions, leading to misleading performance metrics. This can result in load-test dashboards reporting incorrect tail latencies compared to what users experience in production.

In this scenario, the organization’s load test dashboards are showing suboptimal tail latency metrics, while actual production environments exhibit significantly worse performance for extreme latency. This discrepancy could be due to coordinated omission or other factors such as network jitter or resource contention that aren't adequately accounted for during load testing. To address these issues and ensure that p99.9 latency truly reflects user experience in production, we need a robust redesign of both the load generator and the measurement process.

The core idea is to simulate real-world conditions more accurately by addressing potential race conditions and ensuring that all possible paths are processed consistently. This involves understanding how data flows through the system, identifying race conditions, and implementing mechanisms to ensure that no messages or events are omitted during testing.

**Comparison:**

```
Action	Success Rate (%)	Average Latency (ms)
Login	100	~5
Search	100	~3
Purchase	90	~8
```

**Code:**

```python
# Example of a simple load generator function in Python
import random

def simulate_user_actions(num_users):
    for _ in range(num_users):
        action = random.choice(['login', 'search', 'purchase'])
        process_action(action)

def process_action(action):
    # Simulate processing time with race conditions and potential omissions
    if action == 'purchase':
        with purchase_lock:  # Ensuring mutual exclusion
            if can_purchase():  # Check availability
                execute_purchase()  # Perform purchase
```

**Example input / output:**

```text
Input:
- num_users = 100
- purchase_lock acquire time = 5ms, release time = 3ms
- can_purchase availability percentage = 90%, retry attempts = 2

Output (expected):
- Users who attempt to purchase should have a success rate close to 90%
- Latency for successful purchases should be around the sum of lock acquisition and processing times (8ms)
```

**Data:**

```json
{
  "user_actions": ["login", "search", "purchase"],
  "purchase_lock": {
    "acquire_time_ms": 5,
    "release_time_ms": 3
  },
  "can_purchase": {
    "availability_percentage": 90,
    "retry_attempts": 2
  }
}
```

**Flow:**

1. Yes
2. No
3. User Action
4. Is purchase action?
5. Acquire Lock
6. Check Availability
7. Retry?
8. Execute Purchase, Release Lock
9. Simulate other actions

