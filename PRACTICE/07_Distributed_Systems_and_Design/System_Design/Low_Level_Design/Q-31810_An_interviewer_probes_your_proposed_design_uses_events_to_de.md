---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: your proposed design uses events to decouple modules; at 100x scale how do the implicit dependencies and debugging cost of event-driven LLD bite you, and where would you deliberately keep synchronous calls instead?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a Low-Level Design (LLD) for a scalable system, using events to decouple modules can be incredibly powerful in terms of modularity and flexibility. However, as systems scale, especially by an order of magnitude like 100x, the implicit dependencies introduced through event-driven architecture (EDA) start to become non-trivial challenges. Debugging complexity increases, and the overhead of managing these events can introduce costs that may not be apparent in smaller-scale systems. In such scenarios, it's crucial to strategically decide where synchronous calls might still provide a more manageable solution.

Firstly, let's quickly review what event-driven architecture entails: Events are messages passed between different components of the system. These messages are typically asynchronous and handled by event listeners or handlers. The benefits include loose coupling, scalability, and the ability to perform complex operations in parallel. However, this design choice comes with trade-offs that become more apparent as the scale increases.

#### Triaging the Question

The interviewer is probing into the nuances of EDA at a large scale, particularly focusing on:

Implicit Dependencies: Understanding how inter-module dependencies are managed and whether they can lead to complexity.
Debugging Costs: The effort required to diagnose issues when events are involved.
Synchronous Calls as a Deliberate Design Choice: Situations where synchronous calls might be more appropriate due to the benefits they offer in terms of simplicity and ease of debugging.

#### First-Principles Solution
Implicit Dependencies

In an event-driven system, dependencies between modules can be implicit and hard to trace. For instance, if Module A needs data from Module B, it would subscribe to an event that Module B broadcasts. This decouples the modules but complicates dependency management. At scale, this can lead to a complex web of inter-module dependencies that are difficult to understand and manage.

Debugging Costs

Debugging in an event-driven system becomes more challenging because issues can originate from anywhere within the network of events. For example, if a piece of business logic fails due to incorrect event data, identifying the root cause without detailed tracing logs can be time-consuming. The increased complexity of managing these dependencies can lead to a higher debugging cost.

Synchronous Calls

In contrast, synchronous calls provide clear and direct dependency paths. This makes it easier to reason about the flow of execution and trace issues back to their source. However, they can introduce bottlenecks in highly parallel systems where latency is critical.

Strategic Decisions for Synchronous Calls

To strategically decide where to use synchronous calls instead, consider the following factors:

Critical Business Logic: If a piece of business logic is complex or involves multiple steps that must be executed sequentially, synchronous calls can simplify the flow and make it easier to reason about.
Latency Tolerant Operations: In systems where latency is not critical, such as data processing pipelines or batch operations, EDA might introduce unnecessary complexity. Synchronous calls could provide a simpler and more maintainable solution.
Debugging Simplicity: For parts of the system that are prone to frequent errors or require extensive debugging efforts, synchronous calls can make it easier to identify and fix issues.

#### Closing Summary

In summary, while event-driven architecture is powerful for decoupling modules and scaling systems, its implicit dependencies and increased debugging costs become significant at large scale. Strategic use of synchronous calls in critical business logic or latency-tolerant operations can mitigate these challenges and provide a more maintainable design.

#### Common Mistakes/Follow-Ups
Over-relying on Events: Not recognizing situations where synchronous calls are better.
Ignoring Debugging Costs: Underestimating the complexity introduced by event-driven systems.
Neglecting Performance Impact: Failing to consider how event handling can introduce unnecessary latency in critical operations.
Triangulating with Experience

In my experience, the key is finding a balanced approach. Use EDA where it truly provides value but be mindful of its downsides. For instance, in financial trading systems, synchronous calls might be preferred for ensuring accurate order processing, whereas event-driven architecture can excel in handling real-time market data feeds.

**Comparison:**

```
Synchronous Call	Asynchronous Event
Direct dependency path	Indirect dependency through events
Easier to debug	Harder to debug due to event complexity
```

**Code:**

```python
# Example of a simple synchronous call vs asynchronous event handler

def process_order_synchronously(order_data):
    # Simulate order processing logic
    print("Processing order synchronously")
    return True  # Assume successful processing

async def handle_event(order_event):
    # Simulate handling an order event
    print("Handling order event asynchronously")
```

**Example input / output:**

```

Synchronous Call Example Input-Output

CODE
Copy
Input: {"order_id": "12345", "product": "Widget", "quantity": 10, "status": "pending"}
Output: True
Explanation: Order is processed synchronously and the result is immediately available.

Asynchronous Event Handling Example Input-Output

CODE
Copy
Input: {"order_id": "12345", "product": "Widget", "quantity": 10, "status": "pending"}
Event Handled: True
Explanation: Order event is handled asynchronously and the result may not be immediately available.
```

**Data:**

```json
{
  "order_id": "12345",
  "product": "Widget",
  "quantity": 10,
  "status": "pending"
}
```

**Flow:**

1. Yes
2. No
3. Order Processed
4. Is Synchronous?
5. Immediate Result
6. Event Sent

