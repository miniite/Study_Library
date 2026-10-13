---
type: "practice"
difficulty: "Deep"
---

# A single slow dependency (p99 of 2s) is causing your service's threads to all block, cascading into a full outage. Explain how bulkheads and timeouts isolate this, and how you'd right-size the timeout.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, managing slow dependencies can be a critical challenge that leads to full system outages due to thread blocking. A common approach to mitigate such issues is by using bulkheads and timeouts. Bulkheads, which are essentially circuit breakers in the context of dependency management, isolate failing services from the rest of the system, preventing cascading failures. Timeouts ensure that no single request can hold up the entire application for too long.

To address this problem, we'll first triage the key aspects: understanding the current state, identifying the bulkheads and timeouts, and then determining the appropriate timeout duration to right-size our solution.

**Comparison:**

```
Dependency	Current p99 Response Time
ExternalService	2000ms (95th percentile)
InternalService1	500ms
InternalService2	300ms
...	...
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Identify Dependency] --> B{Dependency Response Time};
    B -- Slow dependency(p99 2s) --> C[Implement Bulkhead];
    C --> D[Configure Timeout];
    D --> E[Monitor and Optimize];
```

