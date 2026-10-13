---
type: "practice"
difficulty: "Deep"
---

# Explain the readers-writer lock starvation problem (writers or readers starving). How do fair vs unfair RW locks trade throughput for fairness, and which do you default to in a backend service?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we'll explore the readers-writer lock (RW lock) starvation problem, focusing on how it affects both writers and readers. We'll delve into the trade-offs between fair and unfair RW locks, and discuss which approach is more suitable for backend services.

TL;DR

Readers-writer lock starvation occurs when either writers or readers are starved due to improper handling of the lock protocol. Fairness in RW locks ensures that all threads get a turn, while unfairness optimizes throughput by allowing quicker access but risks causing starvation. In backend services, we typically default to fair locks to ensure data consistency and prevent service degradation.

#### Triaging the Question

The question asks us to explain the readers-writer lock starvation problem, specifically focusing on fairness vs. unfaireness in RW locks, and their implications for a backend service's architecture. We need to understand that starvation can occur due to improper lock handling. The interviewer is probing our understanding of concurrency patterns and the importance of thread scheduling in maintaining system performance.

First Principles: Understanding Readers-Writer Locks

A readers-writer lock allows multiple readers (readers) or one writer (writer) to access a shared resource at the same time, but not both simultaneously. This ensures that no reader can see an inconsistent state while other readers are reading, and no writer is blocked by readers.

The Starvation Problem
Writers Starving Readers

In an unfair RW lock, if many writers queue up behind multiple readers, it can lead to starvation where the writers never get a chance to write. This happens because the lock mechanism might always give higher priority to readers over writers, leading to indefinite delays for the writers.

Readers Starving Writers

Similarly, in scenarios with high reader activity, a writer may be starved if it has to wait indefinitely behind an endless queue of readers who are reading but not releasing the lock. This is less common but still a potential issue.

Fair vs. Unfair RW Locks: Trade-offs
Unfair RW Locks (Default in Many Systems)

Unfair RW locks optimize throughput by giving higher priority to readers, which can lead to faster overall access times for read operations. However, this comes at the cost of fairness and may cause writers to experience prolonged delays.

Fair RW Locks

Fair RW locks ensure that all threads get a chance to proceed in a round-robin fashion. This approach prevents starvation but might reduce throughput because it doesn't favor any specific type of operation (readers or writers).

Defaulting to Fairness in Backend Services

In backend services, we typically default to fair RW locks for the following reasons:

Data Consistency: Ensures that all operations get a chance to complete, maintaining data consistency and integrity.
Avoiding Degradation: Prevents service degradation due to indefinite waits by any type of thread.
User Experience: Provides a more stable user experience as users are less likely to encounter delays.

#### Common Mistakes
and Follow-Ups
Misunderstanding Fairness vs Unfairness: Failing to recognize the implications of unfairness can lead to overlooked potential for starvation.
Ignoring Trade-offs: Overlooking the trade-off between throughput and fairness, which is critical in high-performance systems.
Not Considering Context: Forgetting that context matters; different use cases may require different strategies.

#### Closing Summary

In conclusion, readers-writer lock starvation can occur due to improper handling of lock protocols, leading to either writers or readers getting starved. Fair RW locks ensure fairness but reduce throughput, while unfair RW locks optimize for throughput at the cost of potential starvation. In a backend service context, we default to fair locks to maintain data consistency and avoid service degradation.

Self-Check

Correctness + completeness confirmed.

**Code:**

```
MERMAID
Copy
graph LR;
    subgraph "RW Lock Scenario"
        Writer -->|Write Operation| LockWriter[Lock Writer]
        Reader -->|Read Operation| LockReader[Lock Reader]
        Writer -->|Wait for Readers| QueueWriter[Queue Writers]
        Reader -->|Wait for Writers| QueueReader[Queue Readers]
    end
```

**Data:**

```json
{
  "lockType": "RW Lock",
  "writersQueue": [],
  "readersQueue": []
}
```

