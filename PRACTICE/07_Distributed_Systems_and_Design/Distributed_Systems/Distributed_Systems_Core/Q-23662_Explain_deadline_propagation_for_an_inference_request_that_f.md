---
type: "practice"
difficulty: "Deep"
---

# Explain deadline propagation for an inference request that fans out to a feature store, a ranking model, and a re-ranker. Why should each hop respect the remaining deadline rather than reset its own timeout, and what wasted GPU work results if it doesn't?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we are dealing with an inference request that needs to process data through a feature store, then through a ranking model, and finally through a re-ranker. Each of these components must respect the remaining deadline from the previous stage rather than resetting their own timeout. This ensures efficient use of resources like GPUs and prevents unnecessary delays.

#### Triage
The question is asking about the mechanism for deadline propagation in a distributed system context. Specifically, it's probing why respecting deadlines across different stages (or "hops") is crucial to avoid wasted computational resources. We need to consider how deadlines are enforced at each stage and what happens if they are not respected.

#### First-Principles Solution
Let’s break down the components:

Inference Request: The initial request comes in, with a specified deadline.
Feature Store Hops:
Each hop must respect the remaining deadline from the previous hop.
If a hop resets its own timeout, it could potentially exceed the original deadline, causing delays and wasted resources.
Ranking Model: Similarly, this model should also respect any remaining deadlines to ensure timely processing of requests.
Re-ranker Hops:
The re-ranker components must follow suit by respecting their inherited deadline.
Why Respect Remaining Deadline
Efficient Resource Utilization: By respecting the remaining deadline, each component ensures that it completes its part within a predefined time frame, allowing for efficient use of computational resources like GPUs.
Avoidance of Stale Data: Respecting deadlines helps in ensuring that data is processed in real-time, reducing the risk of using stale or outdated information.
Why Not Resetting Deadlines Causes Wasted GPU Work

If any hop were to reset its own timeout and exceed the original deadline:

Increased Latency: The overall response time for the request increases, leading to slower service times.
Resource Overutilization: Exceeding deadlines can result in overloading of servers or GPUs, which are then underutilized for other tasks until the current task completes.
CPU and Memory Overhead: Extra processing time means increased CPU and memory usage, potentially leading to resource contention with other processes.

#### Example Scenario

Let’s consider a request with an initial deadline of 10 seconds:

Feature Store Hop 1 gets data from the feature store in 2 seconds.
It then passes this data to the Ranking Model, which respects its remaining 8-second deadline and processes it quickly (say, 4 seconds).
The ranking model then forwards the results to the first re-ranker hop, which also has a 4-second deadline. If this hop resets its own timeout to 10 seconds, it might take 6 more seconds, causing an overall delay of 20 seconds instead of the original 10.

#### Closing Summary

Respecting deadlines across all stages in a distributed system ensures efficient use of resources and timely processing. Failing to do so can lead to increased latency, resource overutilization, and unnecessary CPU/Memory overhead.

#### Common Mistakes/Follow-Ups

Misunderstanding Deadline Propagation: Not recognizing the importance of passing deadlines through each stage.
Overconfidence in Local Deadlines: Assuming that local timeouts are sufficient without considering the impact on overall request processing times.
Failure to Monitor and Enforce Deadlines: Lack of mechanisms to monitor or enforce deadlines, leading to unpredictable delays.

**Comparison:**

```
Stage	Time Spent (s)	Remaining Time
Feature Store Hops 1	2	8
Feature Store Hops 2	3	5
Ranking Model	4	1
Re-ranker HopA	1	0
Re-ranker HopB	2	-1
```

**Code:**

```python
# Example pseudo-code for deadline propagation

def process_request(request, initial_deadline):
    current_deadline = initial_deadline

    # Feature Store Hops
    if process_feature_store(request, current_deadline):
        remaining_time = current_deadline - time_spent
        pass_data_to_next_stage(remaining_time)

    # Ranking Model
    if process_ranking_model(request, remaining_time):
        pass_data_to_re_ranker(remaining_time)

    # Re-ranker Hops
    for re_ranker in re_rankers:
        if process_re_ranker(request, remaining_time):
            current_deadline -= time_spent  # Update deadline based on actual processing time

# Pseudocode function to simulate a step
def pass_data_to_next_stage(remaining_time):
    # Process data and update the remaining time
```

**Example input / output:**

```text
Input:
- Request with deadline of 10 seconds

Process Feature Store Hops (remaining time = 10 - 2 = 8)
- Hop1 processes in 2s, remaining_time = 6s
- Hop2 processes in 3s, remaining_time = 3s

Pass data to Ranking Model with 3 seconds remaining.
Ranking model processes in 4s. Remaining time = 3s - 4s (timeout).

Pass data to Re-ranker Hops (remaining time = 3)
- Re-ranker HopA processes in 1s, remaining_time = 2s
- Re-ranker HopB processes in 2s, remaining_time = 0

Output: Result processed within the initial deadline of 10 seconds.
```

**Data:**

```json
{
  "request": {
    "data": "example_data",
    "deadline_seconds": 10
  },
  "feature_store_hops": [
    {"name": "hop1", "time_spent_seconds": 2},
    {"name": "hop2", "time_spent_seconds": 3}
  ],
  "ranking_model": {
    "time_spent_seconds": 4
  },
  "re_ranker_hops": [
    {"name": "hopA", "time_spent_seconds": 1},
    {"name": "hopB", "time_spent_seconds": 2}
  ]
}
```

**Flow:**

1. ReRankerHops
2. FeatureStoreHops
3. Initial Request
4. Feature Store Hops
5. Ranking Model
6. Re-ranker Hops
7. Final Response
8. Hop 1 (2s)
9. Hop 2 (3s)
10. Re-ranker Hop A (1s)
11. Re-ranker Hop B (2s)
12. B1

