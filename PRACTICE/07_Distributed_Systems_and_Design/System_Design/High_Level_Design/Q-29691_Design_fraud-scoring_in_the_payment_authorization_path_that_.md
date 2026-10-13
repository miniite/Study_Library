---
type: "practice"
difficulty: "Deep"
---

# Design fraud-scoring in the payment authorization path that must add under 30ms of latency — how do you fetch features rules and velocity counters synchronously and fall back safely if the scorer times out?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a fraud-scoring system in the payment authorization path that adds no more than 30ms of latency, we need to carefully balance performance with reliability. The core challenge is to fetch necessary features, rules, and velocity counters synchronously while ensuring a safe fallback if the scoring process times out.

#### Triage

The question primarily tests your understanding of high-level design (HLD) for a critical system component. It probes your ability to:

Design a system that meets strict latency requirements.
Ensure data consistency and integrity in a distributed environment.
Implement error handling and fallback mechanisms effectively.
Balance synchronous and asynchronous operations.

#### Assumptions
System Environment: We are designing this for a modern payment gateway with a distributed architecture, where features, rules, and counters can be fetched from various sources.
Latency Requirements: The system must operate within 30ms of latency to ensure real-time processing.
Error Handling: Fallback mechanisms need to handle timeouts gracefully without significantly impacting the overall process.

#### First-Principles Solution
Synchronous Fetching with Deadlines:
We will fetch features, rules, and velocity counters synchronously within a 30ms deadline to ensure no delay in processing.
Parallel Processing:
Use concurrency techniques such as threads or asynchronous tasks to perform these fetches concurrently.
Timeout Mechanisms:
Implement strict timeout policies for each fetch operation to avoid deadlocks and ensure timely responses.
Fallback Strategy:
If any fetch operation times out, we will use pre-fetched or cached data (with eventual consistency) to proceed with the authorization process.
Detailed Steps

Initiate Fetch Operations:

Yes
No
Yes
No
Yes
No
Start Authorization
Fetch Features
Feature Fetch Timeout?
Fallback on Cached Data
Fetch Rules
Rule Fetch Timeout?
Fallback on Default Rules
Fetch Velocity Counters
Counter Fetch Timeout?
Use Pre-Fetched Data
Score Transaction

Concurrency Management:

Use thread pools or asynchronous workers to handle concurrent fetches.
Set a timeout of 10ms for each task, allowing up to 3 parallel tasks (since 3 * 10ms = 30ms).

Fallback Mechanisms:

For features and rules, use an in-memory cache with periodic updates from the database or external service.
For velocity counters, maintain a local copy that is updated periodically.

#### Closing Summary

In summary, to design fraud-scoring within strict latency constraints, we need to employ parallel processing, strict timeout mechanisms, and fallback strategies. This ensures timely authorization decisions while maintaining system reliability and performance.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Failing to account for the 30ms constraint can lead to suboptimal solutions.
Synchronous Overhead: Excessive synchronization can introduce unnecessary delays.
Insufficient Caching: Inadequate caching can result in frequent database hits, increasing latency.

Would you like to explore any specific aspect of this design further?

**Comparison:**

```
Feature	Description	Timeout (ms)
Features	Transaction context data	10
Rules	Scoring logic	10
Velocity Counters	Recent transaction frequency	10
Worker	Task	Timeout (ms)
Worker 1	Fetch Features	10
Worker 2	Fetch Rules	10
Worker 3	Fetch Velocity Counters	10
```

**Code:**

```python
# Example of setting up a thread pool for parallel fetches
import concurrent.futures

def fetch_features():
    # Fetch features logic here
    return "features"

def fetch_rules():
    # Fetch rules logic here
    return "rules"

def fetch_velocity_counters():
    # Fetch velocity counters logic here
    return "velocity_count"

# Set up thread pool with 3 workers for parallel execution
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    feature_future = executor.submit(fetch_features)
    rule_future = executor.submit(fetch_rules)
    counter_future = executor.submit(fetch_velocity_counters)

features = feature_future.result(timeout=0.01)  # 10ms timeout
rules = rule_future.result(timeout=0.01)
velocity_count = counter_future.result(timeout=0.01)
```

**Example input / output:**

```text
Input:
- Payment request with transaction details

Output:
- Fraud score decision (approved or declined)
- Optional: Detailed reasons for the decision based on features, rules, and velocity count
```

**Data:**

```json
{
    "features": ["feature1", "feature2"],
    "rules": {
        "rule1": {"weight": 0.5},
        "rule2": {"weight": 0.3}
    },
    "velocity_count": 500
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start Authorization
8. Fetch Features
9. Feature Fetch Timeout?
10. Fallback on Cached Data
11. Fetch Rules
12. Rule Fetch Timeout?
13. Fallback on Default Rules
14. Fetch Velocity Counters
15. Counter Fetch Timeout?
16. Use Pre-Fetched Data
17. Score Transaction

