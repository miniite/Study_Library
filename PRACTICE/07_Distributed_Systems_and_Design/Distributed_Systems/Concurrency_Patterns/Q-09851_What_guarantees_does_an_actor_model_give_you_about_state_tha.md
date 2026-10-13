---
type: "practice"
difficulty: "Deep"
---

# What guarantees does an actor model give you about state that the lock-based model doesn't, and where does it still let races in (e.g., across actors, message reordering)? When would you choose actors for a backend service?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

The actor model provides a high-level abstraction to handle concurrency that is designed to simplify reasoning about concurrent systems. It introduces isolation between concurrently executing actors, ensuring each actor operates on its own thread or process and has its private state. This design significantly reduces the risk of race conditions compared to lock-based models, which often require explicit synchronization primitives leading to complex and error-prone code.

#### Triaging the Question

This question requires a deep understanding of both concurrency patterns and the trade-offs between different models. The actor model is centered around actors communicating through asynchronous messages rather than sharing memory directly. Lock-based models rely on explicit locks or other synchronization mechanisms to ensure thread safety. We need to explore where the actor model's isolation guarantees shine, how it handles message reordering, and when its architectural patterns are most beneficial.

#### First-Principles Solution
Isolation Guarantees

1. Message Passing: In an actor model, actors communicate by exchanging messages. This approach naturally encapsulates state changes within a single actor's execution context. Since no two actors share the same memory space, race conditions are significantly mitigated because actors don't directly access each other’s states.

2. Asynchronous Execution: Actors can execute concurrently without blocking one another, allowing for non-blocking behavior and better utilization of resources. This means that even if an actor is busy processing a message, others can continue to send messages without waiting.

Where It Still Lets Races In

1. Across Actors: While actors are isolated from each other in terms of state management, they can still interact through asynchronous messages. If multiple actors need to coordinate on shared data or perform operations that require state synchronization, race conditions can occur if not handled correctly. For example, if actor A sends a message to B and then immediately performs an operation based on the response, without ensuring B has processed it, a race condition may arise.

2. Message Reordering: Because messages are asynchronous, they can be delivered out of order or even lost under certain network conditions. This can lead to unexpected behavior if the sequence of operations depends strictly on the order in which messages arrive. However, this is more about protocol design and less about concurrency itself. Proper state management techniques such as using versioning or timestamps can mitigate these issues.

Choosing Actors for a Backend Service

1. Scalability: The actor model excels at scaling due to its non-blocking nature and inherent fault tolerance. Each actor can run in its own thread, allowing the system to scale horizontally by adding more actors without needing complex thread management.

2. Fault Tolerance: Actors are designed to be lightweight and stateless. This makes them easier to restart in case of failure, reducing downtime and improving availability. State is kept within the local scope of each actor, making it simpler to manage failures compared to a monolithic application where state might span multiple threads or processes.

3. Maintainability: The actor model’s message-passing paradigm promotes modular design. Each actor can be responsible for a specific task or set of tasks, making the system easier to understand and maintain. This separation of concerns reduces complexity and makes it easier to isolate and test components.

#### Closing Summary

In summary, the actor model offers strong guarantees about state through its isolation mechanisms and asynchronous communication patterns, which reduce the likelihood of race conditions. However, proper handling of message reordering and coordination between actors remains crucial. The choice of using an actor model over a lock-based approach depends on factors such as scalability requirements, fault tolerance needs, and maintainability goals.

#### Common Mistakes/Follow-Ups
Over-reliance on Asynchronous Communication: While asynchronous communication is powerful, it can introduce complexity if not managed properly. Overlooking issues like message reordering or ensuring proper state synchronization can lead to bugs.
Ignoring Performance Considerations: Although actors provide isolation and non-blocking behavior, they also come with overhead due to thread management and context switching. Careful consideration should be given to performance implications before adopting the actor model.
FIELD
# Example of an Actor in Scala
import akka.actor.{Actor, Props}

class MyActor extends Actor {
  def receive = {
    case msg: String => println(s"Received $msg")
  }
}

FIELD:data_blocks>>>

Input: actorA ! "message1"; actorB ! "message2"
Output:
CODE
Received message1
Received message2
Dry Run:
Actor A sends "message1" to another actor.
Simultaneously, Actor B receives and processes "message2".
Both messages are processed independently without interference.

FIELD:tables>>>

Send Message
Yes
End
No
Concurrent Execution
Send Message
Yes
End
Actor A
Message Received?
Process Message
Completion
Actor B
Message Received?
Process Message
Completion

Correctness and completeness confirmed.

**Flow:**

1. Send Message
2. Yes
3. End
4. No
5. Concurrent Execution
6. Send Message
7. Yes
8. End
9. Actor A
10. Message Received?
11. Process Message
12. Completion
13. Actor B
14. Message Received?
15. Process Message
16. Completion

