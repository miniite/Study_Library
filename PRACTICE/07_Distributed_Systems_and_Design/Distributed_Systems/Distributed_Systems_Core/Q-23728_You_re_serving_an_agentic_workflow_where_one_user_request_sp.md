---
type: "practice"
difficulty: "Expert"
---

# You're serving an agentic workflow where one user request spawns many chained LLM and tool calls across services. Explain the distributed-tracing, deadline-budgeting, and partial-failure design so a single slow tool call doesn't blow the whole agent's latency or leave orphaned in-flight sub-requests.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When serving an agentic workflow that spawns multiple chained calls, ensuring distributed tracing, managing deadlines and budgeting, and handling partial failures is crucial to maintain overall latency and avoid orphaned in-flight sub-requests. This involves using a robust trace system, setting up effective deadline management with buffer budgets, and implementing fail-safes that can handle incomplete workflows gracefully.

#### Triaging the Question

The core of this question revolves around three key aspects:

Distributed Tracing: Tracking the execution path of each request across multiple services.
Deadline Budgeting: Ensuring that no single slow call exceeds a budgeted amount, thus avoiding blowing the entire workflow's latency.
Partial Failure Handling: Managing situations where one or more sub-requests fail to complete without leaving orphaned in-flight requests.

#### Assumptions
The system has multiple services each performing specific tasks (LLMs and tools).
Each request spawns multiple chained calls, potentially leading to a complex execution flow.
There is a defined latency budget for the entire workflow.
Step-by-Step First Principles Solution
Distributed Tracing

Distributed tracing involves assigning unique identifiers to each service invocation and propagating these IDs through every linked service call. This allows us to trace the path of an individual request from its inception until its completion, even if it spans multiple services or retries.

Request Initiation: Generate a unique trace ID for the initial user request.
Service Invocation: Each service that receives this request propagates the same trace ID along with the context (e.g., headers) to all downstream calls.
Logging & Metrics: Log and collect metrics at every step, including response times, failure points, and any relevant metadata.
Deadline Budgeting

To manage deadlines effectively, we need to ensure that no single sub-request exceeds a budgeted amount of time or resources, which could otherwise blow the entire workflow's latency.

Define Latency Budgets: Set clear deadlines for each service call based on the overall workflow requirements.
Buffer Budgets: Allocate buffer time (e.g., 5-10% extra) to account for variability and ensure that even slow calls do not exceed the total budgeted time.
Deadline Enforcement: Implement mechanisms like time-outs, retries with exponential backoff, or circuit breakers to handle slow or failing services without violating the overall deadline.
Partial Failure Handling

Handling partial failures requires a strategy to manage incomplete workflows gracefully and ensure that any orphaned in-flight sub-requests are properly terminated or retried as needed.

Graceful Degradation: Design each service call to fail fast but provide partial results if possible, ensuring the workflow can continue even with some sub-tasks failing.
Rollback Mechanisms: For tasks that can be rolled back (e.g., creating a document), implement rollback logic to clean up any partially completed work.
Orphan Request Management: Use a message queue or database to store in-flight requests and their status. If a service fails, attempt to retry the failed task or mark it as orphaned and notify the user of the failure.

#### Closing Summary

By implementing distributed tracing, effective deadline budgeting with buffer times, and robust partial failure handling strategies, we can ensure that even complex agentic workflows remain responsive and reliable. This approach involves careful design at every step to handle variability and potential failures without compromising overall performance.

#### Common Mistakes
& Follow-Ups
Mistake: Ignoring deadline budgeting entirely, leading to a single slow call causing the entire workflow to exceed its latency constraints.

#### Follow-Up
Discuss how using a deadline budget can be dynamically adjusted based on real-time workload and resource availability.
Mistake: Failing to manage partial failures, resulting in orphaned requests or incomplete workflows that may need manual cleanup.

#### Follow-Up
Explore advanced techniques like distributed consensus protocols (e.g., Raft) for managing complex workflows with many sub-tasks.

**Comparison:**

| Trace ID | Service Call ID | Start Time          | End Time            | Status     |
|----------|-----------------|---------------------|--------------------|------------|
| 12345    | A               | 2023-10-01T12:00:00Z| 2023-10-01T12:01:05Z| completed  |
| 12345    | B               | 2023-10-01T12:01:10Z| 2023-10-01T12:01:50Z| failed     |
| 12345    | C               | 2023-10-01T12:01:55Z| -                   | in_progress|

**Code:**

```python
# Example of setting up a deadline budget using Python's asyncio and timeout decorator
import asyncio

def timeout(timeout_duration):
    def wrapper(func):
        async def inner(*args, **kwargs):
            loop = asyncio.get_event_loop()
            task = loop.create_task(func(*args, **kwargs))
            return await asyncio.wait_for(task, timeout=timeout_duration)
        return inner
    return wrapper

@timeout(10)  # Set a 10-second deadline for this function
async def slow_llm_call():
    # Simulate an LLM call that might take up to the defined deadline
    await asyncio.sleep(15)  # This should ideally be handled with proper timeout logic
```

**Example input / output:**

```text
Input:
- User request with trace ID: 12345

Output:
- Service A completes within the deadline budget.
- Service B fails after exceeding its deadline, triggering a rollback mechanism.
- Service C is marked as in progress and retried if necessary.
```

**Data:**

```json
{
  "trace_id": "12345",
  "service_calls": [
    {
      "call_id": "A",
      "start_time": "2023-10-01T12:00:00Z",
      "end_time": "2023-10-01T12:01:05Z",
      "status": "completed"
    },
    {
      "call_id": "B",
      "start_time": "2023-10-01T12:01:10Z",
      "end_time": "2023-10-01T12:01:50Z",
      "status": "failed"
    },
    {
      "call_id": "C",
      "start_time": "2023-10-01T12:01:55Z",
      "end_time": null,
      "status": "in_progress"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Distribute Trace ID
7. Is Task Complete?
8. Complete Workflow
9. Retry or Fail Task
10. Is Workflow Complete?
11. Notify User of Success
12. Orphan Request Management

