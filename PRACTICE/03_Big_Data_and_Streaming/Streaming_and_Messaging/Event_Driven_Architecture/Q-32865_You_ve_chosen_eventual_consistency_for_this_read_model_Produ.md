---
type: "practice"
difficulty: "Deep"
---

# You've chosen eventual consistency for this read model. Product now says a specific screen needs read-your-writes. Do you compromise the whole architecture, add a synchronous fast-path, or push back on the requirement? Defend your choice and its blast radius.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Read Your Writes in Event Systems](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Read_Your_Writes_in_Event_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with event-driven architectures and eventual consistency, read-your-writes (RYW) can be seen as an oxymoron because it implies immediate visibility of write operations to the reader, which is at odds with eventual consistency's core principle of allowing stale reads. However, in a specific scenario where a user interface requires RYW, we have three main choices: compromise the whole architecture, add a synchronous fast-path, or push back on the requirement. Each choice has its trade-offs and implications, so let’s break it down from first principles.

Firstly, we need to understand that our current architecture is built around eventual consistency for various reasons such as scalability, resilience, and performance. Eventual consistency ensures that data will eventually converge to a consistent state across all nodes in the system but allows for temporary discrepancies. Adding RYW would disrupt this balance. If we decide to go down any of these paths, we need to carefully consider the impact on other parts of the system.

If we compromise the whole architecture and introduce immediate consistency everywhere, it might lead to significant changes such as increased complexity, potential deadlocks, or even compromising the overall availability and scalability of our system. This approach is not advisable unless there are no alternatives and the requirement for RYW is absolutely critical across all screens.

Adding a synchronous fast-path could involve introducing a new component or modifying existing ones to handle this specific read operation with immediate consistency. This would mean that for certain operations, we bypass the usual event-driven pattern and use a more direct approach. The downside of this path is increased complexity in maintaining two different paths (eventual and immediate) within our architecture.

Pushing back on the requirement might be the most pragmatic approach if RYW is only needed sporadically or on a few screens. This would mean educating stakeholders about why eventual consistency is beneficial and how it can coexist with occasional read-your-writes operations without fundamentally altering the architecture.

Now, let’s explore each option in more detail to see which one makes the most sense for our scenario:

Compromising the Whole Architecture
Complexity: Introducing immediate consistency everywhere would require redesigning parts of the system to ensure that every write operation is immediately visible.
Impact on Scalability: Ensuring immediate visibility might require additional synchronization mechanisms, which could lead to performance bottlenecks and increased latency.
Risk of Deadlocks: With multiple components needing immediate consistency, there’s an increased risk of deadlocks or other concurrency issues.
Adding a Synchronous Fast-Path
Complexity Management: Designing a fast-path involves additional code paths and state management, which can complicate the overall system.
Resource Utilization: The fast-path might require more resources (e.g., memory, CPU) to handle immediate consistency checks, potentially affecting other parts of the architecture.
Maintainability: Adding complexity in one part of the system can make it harder to maintain and debug.
Pushing Back on the Requirement
Educate Stakeholders: Explain why eventual consistency is beneficial for most use cases and how RYW can be handled with targeted, asynchronous approaches when needed.
Hybrid Approach: Implement a hybrid approach where immediate reads are possible but only in certain conditions or through separate APIs that don’t affect the rest of the system.

Given these considerations, pushing back on the requirement might be the most strategic move unless there is an overwhelming need for RYW across multiple screens. If we must implement RYW, adding a fast-path would likely be the least disruptive approach while still meeting the specific requirement.

#### Closing Summary
and Common Mistakes

The key challenge here is balancing immediate consistency with eventual consistency without fundamentally altering the architecture. Common mistakes include:

Overcomplicating the Architecture: Introducing unnecessary complexity by trying to satisfy RYW requirements everywhere.
Ignoring Performance Implications: Failing to consider how additional synchronization mechanisms impact overall system performance.
Misunderstanding Requirements: Assuming that RYW is needed across all screens when it might only be necessary in a few specific cases.

Defending our choice should focus on the trade-offs and the strategic value of maintaining a consistent architecture while addressing specific needs through targeted solutions.

**Comparison:**

```
Option	Complexity	Impact on Scalability	Resource Utilization	Maintainability
Compromise the Whole Architecture	High	Significant	High	High
Add a Synchronous Fast-Path	Moderate	Low-Moderate	Moderate	Moderate-High
Push Back on the Requirement	Low	Minimal	Low	Low
```

**Code:**

```python
# Example code for implementing a synchronous fast-path

def handle_eventual_read(operation_id):
    # Eventual consistency logic here
    pass

def handle_synchronous_read(operation_id):
    # Immediate consistency logic here
    pass

# Decision point in the event handler
if need_immediate_consistency:
    return handle_synchronous_read(operation_id)
else:
    return handle_eventual_read(operation_id)
```

**Example input / output:**

| Input | Operation Type | Output |
|-------|---------------|--------|
| `handle_eventual_read("12345")` | Eventual Read | Processed with eventual consistency logic |
| `handle_synchronous_read("12345")` | Synchronous Read | Processed with immediate consistency logic |

**Data:**

```json
{
  "operation_id": "12345",
  "data": {
    "event_type": "write",
    "payload": {"key": "value"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Event Occurs
4. Is Immediate Read Needed?
5. Handle Synchronous Read
6. Handle Eventual Read

