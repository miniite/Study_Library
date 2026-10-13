---
type: "practice"
difficulty: "Deep"
---

# Contrast the saga pattern (orchestration vs choreography) with a distributed 2PC transaction. When would you tolerate the eventual-consistency window of a saga, and how do you design compensating actions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, managing transactions across multiple services can be challenging due to network latency, system failures, and inconsistencies. Two common approaches are the saga pattern (which is an example of orchestration) and the 2PC (Two-Phase Commit) transaction model (a form of choreography). This answer will contrast these two patterns, explain when you might tolerate eventual consistency in a saga, and discuss how to design compensating actions.

#### Triage
What’s Being Probed

The question asks for an understanding of distributed systems patterns and the trade-offs between them. The interviewer is probing your ability to reason about transactional behavior in complex environments and how to handle failures gracefully. The assumptions are that you understand both saga patterns and 2PC, as well as their respective strengths and weaknesses.

#### Solution
From First Principles
Saga Pattern (Orchestration)

In a saga, transactions are orchestrated through a series of steps initiated by an external client or service. Each step is a compensating action in case a subsequent step fails. The key idea here is that the saga pattern allows for more granular and flexible handling of failures.

Workflow Structure: A saga consists of multiple operations that must succeed together.
Compensating Actions: If an operation fails, compensating actions are executed to reverse the effects of previous successful operations.
Eventual Consistency: Sagas can tolerate higher eventual consistency because each step is reversible.
2PC (Two-Phase Commit) Transaction

The 2PC transaction model involves two phases:

Prepare Phase: All participating services are asked if they are ready to commit the transaction.

Commit/Abort Phase: Once all participants agree, the transaction commits; otherwise, it aborts.

Atomicity and Consistency: 2PC ensures strong consistency but can introduce significant latency due to the need for coordination across nodes.

Failure Handling: If a participant fails during the prepare phase, the transaction must be aborted, leading to potential deadlocks or retries.

When Would You Tolerate the Eventual-Consistency Window of a Saga?

You would tolerate the eventual consistency window in sagas when:

Latency Sensitivity is Low: The application can afford some delay before seeing all changes.
Complexity and Flexibility are Needed: Operations must be reversible, and complex business logic requires detailed control over each step.
Scalability Requirements: Sagas can scale more easily because they do not require central coordination.
Designing Compensating Actions

Compensating actions in sagas should:

Be Simple and Reversible: Each action must be reversible, meaning you can undo its effects if a failure occurs.
Avoid Circular Dependencies: Ensure that no service depends on another’s compensating action to avoid creating complex interdependencies.
Error Handling: Implement robust error handling to manage retries and failures gracefully.

#### Closing Summary

Sagas provide a flexible approach to managing transactions in distributed systems by allowing for granular control over each step, including compensating actions. The eventual consistency model of sagas is suitable when strong consistency is not as critical, and complex business logic necessitates fine-grained transaction management. In contrast, 2PC ensures atomicity and consistency but comes with higher latency and more complexity in failure handling.

#### Common Mistakes
and Follow-Ups
Overlooking Compensating Actions: Failing to design compensating actions can lead to irrecoverable states.
Ignoring Latency: Underestimating the impact of network delays can result in poor user experience.
Misunderstanding Eventual Consistency: Thinking that eventual consistency means no data is lost, which isn’t always true.

#### Follow-ups
could involve discussing specific design patterns for implementing compensating actions, or comparing sagas with other transaction models like linearizable transactions.

