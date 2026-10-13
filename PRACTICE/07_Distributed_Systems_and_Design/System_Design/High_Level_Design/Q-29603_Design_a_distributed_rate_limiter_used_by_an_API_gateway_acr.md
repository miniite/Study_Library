---
type: "practice"
difficulty: "Deep"
---

# Design a distributed rate limiter used by an API gateway across 50 nodes — compare token bucket, leaky bucket, and sliding-window-log, and where you store the state.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed rate limiter that can handle requests from 50 nodes in an API gateway scenario, we need to consider both efficiency and consistency across multiple nodes. The key concepts here are token bucket, leaky bucket, and sliding-window-log algorithms. Each has its own trade-offs, and the choice of which to use depends on specific requirements like latency tolerance and state management.

#### Triage
What's Being Probed

The interviewer is likely probing your understanding of rate limiting algorithms, distributed systems, and state management strategies. Assumptions include that requests are not too frequent (e.g., each node sends a limited number of requests per second) to avoid overwhelming the system, and that the API gateway needs to handle a significant volume of traffic.

#### First-Principles Solution

Token Bucket Algorithm:

Concept: Tokens are added at a constant rate but can be consumed more rapidly when needed.

#### Implementation
Each node generates tokens at a fixed rate and sends requests until the token bucket is empty, allowing bursts of high-rate requests.

#### Pros
Handles bursty traffic well; easy to implement in distributed systems using local state.

#### Cons
Potential for unfairness if tokens are added too slowly.

Leaky Bucket Algorithm:

Concept: Tokens leak out at a constant rate, and excess tokens accumulate but do not overflow.

#### Implementation
Each node sends requests until the bucket is empty, allowing steady traffic but no bursts.

#### Pros
Simple to implement; ensures consistent rate control.

#### Cons
Does not handle bursty traffic well; state management can be complex.

Sliding-Window-Log Algorithm:

Concept: Maintains a sliding window of recent requests and counts them to enforce the limit.

#### Implementation
Uses a log-based approach where each request increments an integer in the log, and when the counter exceeds the rate limit, further requests are throttled.

#### Pros
Efficient for large-scale systems; handles both bursts and steady traffic well.

#### Cons
Requires distributed state management, which can introduce complexity.
State Management Considerations
Local vs. Distributed State: Local state (e.g., token bucket) is easier to implement but may lead to inconsistencies across nodes. Distributed state ensures consistency but increases the complexity of state synchronization and failure recovery.
Failure Recovery: In a distributed system, failures can cause stale or corrupted state data. Strategies like leader election for state updates can help mitigate this.
Full Solution

Given 50 nodes, we would implement a hybrid approach using sliding-window-log with distributed state management:

State Storage:

Use a distributed database (e.g., Redis) to store the sliding window log and current rate limit counters.
Nodes write their request counts to the shared state on each request.

Rate Limit Enforcement:

Each node maintains a local copy of the global state and updates it based on received requests.
When a new request arrives, the node checks the local state to see if the rate limit has been exceeded.
If the limit is not exceeded, the request is processed; otherwise, it is throttled.

State Synchronization:

Implement a leader election algorithm (e.g., Raft) to manage updates to the shared state.
All nodes regularly sync their local copies with the current global state to ensure consistency.

#### Common Mistakes
and Follow-Ups
Mistake: Focusing solely on one rate limiting algorithm without considering the trade-offs. Always compare multiple algorithms based on system requirements.

#### Follow-Up
Discussing the implications of different failure modes, such as network partitions or node failures in a distributed setup.
Additional Depth: Exploring advanced topics like machine learning-based rate limiting to dynamically adjust limits based on real-time traffic patterns.

**Comparison:**

| Algorithm | Pros | Cons |
|-----------|------|------|
| Token Bucket | Handles bursty traffic well; easy to implement with local state. | Potential for unfairness due to slow token refill rate. |
| Leaky Bucket | Simple implementation; ensures steady traffic flow. | Does not handle bursts; complex state management. |
| Sliding Window Log | Efficient for large-scale systems; handles both bursts and steady traffic. | Requires distributed state management, increasing complexity. |

**Code:**

```
MERMAID
Copy
graph TB
    Node1[Node 1] -->|Generate Token| TokenBucketState
    Node2[Node 2] -->|Send Request| LeakyBucketState
    Node3[Node 3] -->|Log Request| SlidingWindowLogState
```

**Example input / output:**

```text
Input:
Node 1 sends a request to the API gateway.

Output:
- TokenBucketState: Tokens = 49, Refill Rate = 10
- LeakyBucketState: Bucket Level = 45, Leaky Rate = 5
- SlidingWindowLogState: Request Count = 300

Decision:
Token Bucket - Process the request.
Leaky Bucket - Check bucket level; if > 0, process; else throttle.
Sliding Window Log - Check request count in current window; if < limit, process; else throttle.
```

**Data:**

```yaml
# Example of state data structure in Redis
api-rate-limiter:
  token-bucket-state: {node_id: "123", tokens: 50, refill_rate: 10}
  leaky-bucket-state: {node_id: "123", bucket_level: 45}
  sliding-window-log-state: {node_id: "123", request_count: 300}
```

**Flow:**

1. | ProcessRequest[ProcessRequest]Node3[Node 3]
2. Yes
3. No
4. Node 1
5. Generate Token
6. Refill Bucket at fixed rate
7. Update Local State
8. Node 2
9. Send Request to API Gateway
10. Check Leaky Bucket Level
11. Log Request in Sliding WindowLog
12. Count Requests in CurrentWindow
13. Within Rate Limit?
14. ProcessRequest
15. ThrottleRequest

