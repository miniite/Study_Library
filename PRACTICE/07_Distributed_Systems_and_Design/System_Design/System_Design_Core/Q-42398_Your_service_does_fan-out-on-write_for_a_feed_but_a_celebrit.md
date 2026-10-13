---
type: "practice"
difficulty: "Expert"
---

# Your service does fan-out-on-write for a feed, but a celebrity with 40M followers now stalls the write path for everyone. How would you evolve to a hybrid push-pull model, and what's the read-time cost?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the issue of a celebrity stalling the write path in a fan-out-on-write system, we need to evolve our architecture to a hybrid push-pull model. This approach combines the scalability benefits of push notifications with the efficiency and consistency of pull-based reads. The key idea is to separate the write path into two distinct processes: one that pushes updates directly to subscribers (push) and another that pulls updates from a central queue or database (pull). Let's break down the solution step by step.

Identify the Problem: A celebrity with 40M followers stalls the write path, causing delays in updating all followers.
Understand What is Being Probed: The interviewer is testing your ability to handle high-traffic scenarios, manage concurrency and load balancing, and design a robust system that can scale horizontally.

#### Triaging the Question

The question requires us to redesign our system so that it can efficiently handle a large number of writes without stalling. A push-pull model would help by decoupling the write path from the read path. Here are the assumptions:

The system processes a significant amount of data.
We need to ensure high availability and minimal delays in delivering updates.
Step-by-Step Solution

Centralized Write Queue:

Implement a message queue (e.g., Kafka, RabbitMQ) to buffer writes from the main service. This will help in managing write operations more efficiently by decoupling them.

Push Mechanism for Subscribers:

Use a pub/sub model where subscribers can opt-in and out of receiving updates. For celebrities with many followers, push notifications are sent directly to their followers' feed caches or databases.

Pull Mechanism for Less Active Users:

For users who are not as active (e.g., those with fewer followers), use a pull mechanism where they periodically fetch the latest updates from the central database or cache.
Detailed Implementation

Message Queue Setup:

Subscriber
Service
Send
Push/Pull
Main Service
Write Queue
Cache/DB

Push Mechanism:

When a write operation occurs (e.g., posting a new update), it is first sent to the message queue.
For celebrities, their followers are registered as subscribers in a push-based model.
The system pushes updates directly to these subscribers' caches or databases.

Pull Mechanism:

For less active users, the system pulls the latest updates from the central cache or database.
This reduces the load on the main service and ensures that non-celebrities get their updates more efficiently.

#### Closing Summary

The hybrid push-pull model enhances our ability to handle high traffic by separating write operations from read operations. Push notifications ensure that celebrities with a large following receive updates quickly, while pull mechanisms manage less active users more efficiently. This approach minimizes the risk of bottlenecks and ensures a better user experience.

#### Common Mistakes/Follow-Ups
Misunderstanding Queue Mechanism: Not properly understanding how to use a message queue effectively.
Ignoring Scalability Issues: Failing to address scalability concerns for both push and pull mechanisms.
Inadequate Caching Strategy: Poorly designed caching strategies can lead to inconsistencies or increased load.

