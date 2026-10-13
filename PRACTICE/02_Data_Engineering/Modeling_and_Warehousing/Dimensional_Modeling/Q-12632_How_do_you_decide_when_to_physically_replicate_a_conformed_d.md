---
type: "practice"
difficulty: "Deep"
---

# How do you decide when to physically replicate a conformed dimension into each domain mart versus keeping one shared copy, given the consistency and coupling trade-offs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to physically replicate a conformed dimension into each domain mart or keep it as a single shared copy, we need to weigh the trade-offs in terms of data consistency, operational complexity, and performance. The key factors include the size and frequency of changes to dimensions, the number of marts that depend on these dimensions, and the read versus write access patterns.

#### Triaging the Question

The question is probing our understanding of how to design efficient and scalable data models in a dimensional modeling context, particularly focusing on conformed dimensions. We need to consider the trade-offs between consistency (ensuring all users see the same version of truth) and coupling (reducing the complexity of queries by minimizing dependencies).

#### First-Principles Solution

Dimension Size and Frequency of Changes:

If the dimension is small or rarely changes, replicating it may not significantly impact performance.
Conversely, large dimensions with frequent updates can lead to data inconsistencies if managed incorrectly.

Number of Marts Using the Dimension:

A single shared copy reduces coupling but can increase query complexity if many marts need to access a common dimension.
Multiple copies allow each mart to maintain local consistency and reduce network latency, which is beneficial for performance.

Read vs Write Access Patterns:

High read access on dimensions with low write frequency may benefit from replication to reduce contention.
Frequent writes or updates to the same dimension across marts suggest a shared copy might be more appropriate.

#### Cons
istency Requirements:

Real-time applications often require high consistency and can tolerate increased complexity in managing replicated data.
Batch processing systems might allow some degree of inconsistency during periods of heavy write activity, favoring replication for performance reasons.

Technological Considerations:

Modern database technologies support distributed transactions or materialized views that could mitigate the need for multiple copies.
Replication mechanisms like log-based replication can help maintain consistency without duplicating data storage.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the decision hinges on balancing consistency with performance. A shared dimension might be more appropriate in scenarios with frequent writes across marts, while replication is beneficial when read-heavy operations or real-time consistency are critical. Always validate your choice through prototyping and stress testing.

#### Common Mistakes

Overlooking the frequency of changes and assuming a one-size-fits-all approach.
Ignoring the impact of network latency on performance in distributed systems.

#### Follow-Ups

What specific technologies do you have available for managing consistency?
How would this decision change with different scaling requirements?

**Code:**

```
MERMAID
Copy
graph TD;
    A[Dimension Size and Frequency] --> B{Is the dimension small or rarely changing?}
    B -- Yes --> C[Replicate Dimension]
    B -- No --> D{Do many marts use this dimension frequently?}
    D -- Yes --> E[Shared Copy for Simplicity]
    D -- No --> F[Replicate for Performance]

    A --> G{Does the application require real-time consistency?}
    G -- Yes --> C
    G -- No --> H[Consider Replication or Distributed Transactions]
```

