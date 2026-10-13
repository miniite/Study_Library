---
type: "practice"
difficulty: "Deep"
---

# Mentor a senior engineer who insists exactly-once is always the safest default. Teach them the mental model for reasoning about delivery semantics per use case, and give them a decision tree they can apply without you.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we first need to understand that exactly-once message processing is indeed safer in many scenarios due to its guarantees of no duplicates or missed messages. However, it's not always the best default, as there are performance and complexity trade-offs involved. The key is to develop a mental model for reasoning about delivery semantics based on specific use cases.

#### Triage
What the Question is Asking

The senior engineer insists that exactly-once is the safest default. They need to understand why this might not always be true and learn a decision-making process that can help them choose between at-least-once and exactly-once delivery semantics based on the requirements of each use case.

#### Assumptions
We are working in a distributed system where messages must be processed reliably.
Exactly-once is ideal for ensuring no duplicates or missed messages, but it comes with significant complexity.
At-least-once ensures all messages are delivered at least once, making it easier to implement and more performant.

#### First-Principles Solution

Identify the Use Case:

Understand the business requirements, such as data integrity, latency, and throughput.

Define the Delivery Semantics:

At-least-once: Ensures messages are delivered at least once (no duplicates).
Exactly-once: Ensures each message is processed exactly once (no duplicates or missed messages).

Reasoning Steps:

Step 1: Business Requirements
Data Integrity: If data integrity is critical, and you cannot afford any duplicate or missing records, choose exactly-once.
Latency Constraints: If minimal latency is crucial, consider at-least-once as it is more performant. Exactly-once can add significant overhead to ensure no duplicates.
Step 2: Technical Feasibility

At-Least-Once:

Relatively simple to implement using acknowledgments and retries.
Suitable for scenarios where occasional duplicates are acceptable, but missing data is not.

Exactly-Once:

Requires strong consistency guarantees.
Involves complex transactional logic, such as two-phase commit protocols or distributed transactions.
Higher performance overhead due to additional checks and balances.
Step 3: Trade-offs
Performance vs. Complexity: At-least-once is simpler and more performant but might introduce duplicates. Exactly-once ensures no duplicates but can be complex and costly in terms of performance.
Error Handling: At-least-once systems require robust error handling to manage retries, while exactly-once requires additional mechanisms like compensating actions.
Step 4: Decision Tree
Critical Data Integrity?
Yes: Go with exactly-once.
No: Consider at-least-once.
Latency Requirements?
High: Prefer at-least-once if acceptable risk of duplicates.
Low: Can afford additional latency for exactly-once guarantees.

#### Closing Summary

The decision between at-least-once and exactly-once depends on the specific use case, balancing data integrity with performance. Exactly-once is ideal when no duplicates or missed messages are tolerable, but it comes with increased complexity and performance overhead. At-least-once offers a simpler implementation and better performance in scenarios where occasional duplicates can be managed.

#### Common Mistakes/Follow-Ups
Misunderstanding Latency: Overlooking the impact of additional latency on user experience.
Ignoring Data Integrity: Failing to recognize that exactly-once is crucial for certain types of data processing, such as financial transactions.
Overestimating Simplicity: Assuming at-least-once will be straightforward when handling large volumes or critical systems.

By walking through this decision tree, the senior engineer can make informed choices based on the specific requirements and constraints of their use case.

