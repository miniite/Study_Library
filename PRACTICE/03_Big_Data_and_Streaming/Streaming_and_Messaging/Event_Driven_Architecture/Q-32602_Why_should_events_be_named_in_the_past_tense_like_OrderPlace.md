---
type: "practice"
difficulty: "Basic"
---

# Why should events be named in the past tense, like OrderPlaced instead of PlaceOrder?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system using message queues and event-driven architecture (EDA), it's common to name events in the past tense, such as OrderPlaced rather than PlaceOrder. This naming convention has several advantages that make the overall architecture more robust, maintainable, and easier to reason about.

#### Triage

The question is essentially asking for an explanation of why using past tense event names like OrderPlaced is preferred over present tense names such as PlaceOrder. The interviewer might be probing your understanding of EDA principles, system design, and how naming conventions can influence the behavior and robustness of a system.

#### First-Principles Solution

#### Cons
istency with Event Behavior: Events in an event-driven architecture represent completed actions or states that have already occurred. Naming them in past tense aligns with this reality.
Historical Semantics: Past tense names help maintain historical context, making it clear when the action was performed and ensuring that the system is thinking about what has happened rather than what is happening or will happen.
Asynchronous Processing: In EDA, events are often processed asynchronously by different components in the system. Using past tense ensures that consumers of these events understand they can rely on their occurrence without having to worry about concurrent operations.
Detailed Explanation

#### Cons
istency and Clarity: Past tense names like OrderPlaced make it clear that the action is complete. This reduces confusion, especially for developers who might not be aware of real-time processing nuances.
Robustness in Asynchronous Systems: In an asynchronous system, events need to be treated as historical facts. The term OrderPlaced implies that an order has been placed and successfully processed by the backend, which can then be acted upon without worrying about ongoing or concurrent operations.
Avoiding Race Conditions: Using present tense names like PlaceOrder might lead to misunderstandings if a consumer reads the event mid-execution. For instance, PlaceOrder could imply that it's still in progress and the system should wait, which can create unnecessary delays or even deadlocks.
Practical Example

Let’s consider an example where we have two microservices: one responsible for placing orders and another for processing payments:

PLAINTEXT
[OrderService] -> [PaymentService]

Past Tense Event: OrderPlaced

When OrderService sends a OrderPlaced event to PaymentService, it indicates that the order has already been created.
PaymentService can safely process this event without concern for any ongoing placement of the order.

Present Tense Event: PlaceOrder

If using PlaceOrder, there might be confusion if another microservice reads this event mid-execution. It could lead to unnecessary delays or incorrect processing logic because it’s unclear whether the order is still being placed.

#### Common Mistakes
and Follow-Ups
Misunderstanding Event Tenses: Sometimes developers might use present tense events like PlaceOrder without realizing that they should be in past tense (OrderPlaced). This can lead to subtle bugs where systems misinterpret event states.
Overthinking vs. Simplicity: While it's important to name events correctly, overly complex naming conventions can obscure the simplicity of EDA principles.

#### Closing Summary

Naming events in the past tense is a best practice in message queues and event-driven architectures because it aligns with the historical nature of these events, enhances system robustness, and simplifies reasoning about asynchronous processing. Always consider how your choice of words impacts the clarity and reliability of your system's behavior.

**Code:**

```
MERMAID
Copy
graph LR;
    A[OrderService] --> B(OrderPlaced);
    B --> C[PaymentService];
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
[OrderService] -> [PaymentService]
Event: OrderPlaced at 2023-10-04T15:00:00Z

Output:

PLAINTEXT
Copy
[PaymentService] Processed OrderPlaced event.
```

**Data:**

```json
{
  "event": {
    "type": "OrderPlaced",
    "timestamp": "2023-10-04T15:00:00Z"
  }
}
```

