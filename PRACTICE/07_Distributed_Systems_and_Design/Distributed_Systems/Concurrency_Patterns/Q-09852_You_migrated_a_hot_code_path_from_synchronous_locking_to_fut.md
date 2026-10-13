---
type: "practice"
difficulty: "Expert"
---

# You migrated a hot code path from synchronous locking to futures/async composition and latency got worse under load. What are the likely causes and how do you profile async code where the stack trace no longer tells the story?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a high-performance backend system, migrating from synchronous locking to futures or async composition can significantly impact performance if not done correctly. The primary causes of increased latency after such a migration often relate to improper handling of concurrency and race conditions. This scenario requires a detailed investigation into how the new asynchronous model is managing resources, task scheduling, and communication between threads.

To address this issue, we need to understand that in an async system, stack traces no longer provide a linear view of execution flow due to the asynchronous nature of tasks. We must rely on other profiling techniques such as performance counters, tracing tools, and analysis of logs. The goal is to identify bottlenecks and misbehaviors caused by race conditions or improper resource management.

#### Triage
What's Probing and Assumptions

The question is probing our understanding of async programming patterns, concurrency issues like race conditions, and the challenges in profiling asynchronous code. Our assumptions include:

The system uses a modern async framework (e.g., Node.js with Promises, Python with asyncio).
We have access to logging infrastructure.
Profiling tools are available for the platform.

#### First Principles Solution

Identify Synchronization Points: Analyze the original synchronous code path to understand where contention and locking occurred. This helps in determining what needs to be replaced or adjusted in the async version.

Review Async Patterns: Ensure that futures, promises, or coroutines are used correctly. Common mistakes include:

Using async functions incorrectly (e.g., blocking inside an async function).
Improper resource management (e.g., not closing connections or files properly).
Inadequate error handling.

Profile Async Code:

Event Loop Analysis: Use tools like Node.js’s --inspect-brk, Python’s tracemalloc, or specialized async profilers to understand the event loop behavior.
Tracing and Logging: Implement detailed logging at key points in your async flow to capture task execution timelines and identify where delays occur.
Performance Counters: Utilize system-level metrics (e.g., CPU, memory usage) and language-specific profiling tools (e.g., Node.js’s perf_hooks).

Race Condition Detection:

Deadlocks and Starvation: Check for scenarios where tasks are waiting indefinitely or being starved of resources.
Concurrent Access: Ensure that shared state is accessed safely using locks, semaphores, or other synchronization primitives.

Optimize Communication Patterns: Analyze how async tasks communicate with each other:

Asynchronous Context Propagation: Ensure that context (like session data) is passed correctly between asynchronous calls.
Concurrency Limits: Implement limits to prevent overwhelming the system with too many concurrent requests.

Test and Validate:

Use load testing tools like Apache JMeter or K6 to simulate real-world traffic patterns and observe the impact on performance.
Validate assumptions by running tests in both synchronous and async modes under similar conditions.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully analyzing the original synchronous code, understanding the nuances of asynchronous programming, and leveraging profiling tools effectively, we can pinpoint and resolve the issues causing increased latency. Common pitfalls include misuse of async patterns, improper resource management, and failure to handle race conditions properly. Follow-ups might involve revisiting the design choices or exploring alternative concurrency models.

**Comparison:**

```
Metric	Value
CPU Usage (%)	75.00
Memory Usage (MB)	450.00
Network I/O (KB/s)	600.00
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Start] --> B{Is synchronous code path identified?};
    B -- Yes --> C[Review sync patterns];
    B -- No --> D[Identify synchronization points in async path];
    C --> E[Profile async code using tools];
    D --> F{Detect race conditions?};
    F -- Yes --> G[Optimize communication and concurrency limits];
    F -- No --> H[Test under load];
    G --> I[Validate assumptions];
    H --> J[Continue testing and refining];
    I --> J;
```

**Example input / output:**

```text
Request: GET /data
Response: {"status": "success", "data": [1, 2, 3]}
```

**Data:**

```yaml
# Example YAML for logging configuration
logging:
  level: debug
  handlers:
    - type: file
      filename: async.log
      level: debug
      formatter: full
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Identify Synchronization Points
7. Review Sync Patterns
8. Analyze Async Code
9. Profile and Trace
10. Detect Race Conditions?
11. Optimize CommunicationPatterns
12. Test Under Load
13. Continue Testing
14. Validate Assumptions

