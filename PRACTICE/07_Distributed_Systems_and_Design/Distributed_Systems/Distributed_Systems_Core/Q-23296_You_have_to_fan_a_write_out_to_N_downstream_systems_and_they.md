---
type: "practice"
difficulty: "Deep"
---

# You have to fan a write out to N downstream systems and they must all eventually reflect it, but some are flaky. Compare a queue-per-consumer, an outbox+CDC, and a workflow engine for this fan-out.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing distributed writes across multiple downstream systems in a scenario where some of these systems may be flaky, we must consider reliability, consistency, and fault tolerance. Let's compare three approaches: a queue-per-consumer, an outbox with Change Data Capture (CDC), and a workflow engine.

Firstly, the core question is to evaluate which approach best meets our needs for ensuring that all downstream systems eventually reflect the write operation despite potential flakiness in some consumers. Each method has unique trade-offs, particularly around reliability, complexity, and scalability.

**Example input / output:**

```text
Input:
- Write operation to be sent to `N` downstream systems.

Output (for each approach):
- Queue-per-consumer
  - Each system has its own queue.
  - A message is placed in the corresponding queue for that system.
- Outbox with CDC
  - A central outbox captures write operations.
  - Changes are propagated using a changelog or CDC mechanism to downstream systems.
- Workflow engine
  - Orchestrates steps involving coordination across multiple services.
  - Ensures idempotence and retries with conditions.
```

**Data:**

```json
{
  "downstream_systems": ["sys1", "sys2", "sys3"],
  "flaky_probability": {"sys2": 0.1, "sys3": 0.2}
}
```

