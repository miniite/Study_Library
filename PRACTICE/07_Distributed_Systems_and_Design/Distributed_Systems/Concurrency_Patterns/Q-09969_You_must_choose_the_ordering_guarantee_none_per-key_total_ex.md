---
type: "practice"
difficulty: "Expert"
---

# You must choose the ordering guarantee (none / per-key / total) exposed by a shared event bus that 50 teams build on. Each stronger guarantee costs throughput and operational complexity. How do you pick the org-wide default and let teams opt up, and why not just give everyone total order?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a shared event bus that 50 teams will build on, choosing the right ordering guarantee is crucial to balancing performance, complexity, and reliability. Concurrency patterns such as none, per-key, or total order each have trade-offs. As an expert backend architect, you need to weigh these factors against your organization's needs and risk tolerance.

The key goal here is to provide a flexible default while allowing teams to opt-in for stronger guarantees when necessary. The decision involves understanding the implications of each ordering guarantee on throughput, complexity, and potential data inconsistencies. Given that total order costs the most in terms of both performance and operational overhead, it’s not advisable to set this as the default.

#### Triage
Question: Choose the organizational default for a shared event bus with 50 teams using concurrency patterns (none, per-key, or total order).

#### Assumptions
Teams are diverse in their requirements.
The event bus must support various use cases.
Throughput and operational complexity are significant concerns.

#### Solution
Step-by-Step Reasoning

Understanding Concurrency Patterns:

None: No ordering guarantee; events can arrive at any order or may be dropped if the system is under high load.
Per-key: Events with the same key (e.g., user ID) are delivered in order, but not necessarily globally ordered.
Total Order: All events are delivered strictly in order they were produced.

Impact Analysis:

None Order: Least complex and highest throughput. Suitable for low-latency, high-frequency applications where minor inconsistencies can be tolerated.
Per-key Order: Moderate complexity and throughput. Useful when events need to be processed sequentially per key but not globally. Can handle many more teams than total order due to the reduced global ordering constraint.
Total Order: Highest complexity and lowest throughput. Best suited for scenarios requiring strict temporal ordering, such as financial transactions or critical business processes.

Opting Up:

Allow teams with specific requirements to opt into stronger guarantees while keeping the default flexible. For instance, a team working on real-time notifications might prefer none order due to its high throughput.
Provide clear documentation and guidelines for teams opting up, ensuring they understand the implications of their choice.

Risk Management:

Evaluate the potential impact of each ordering guarantee on the overall system stability.

#### Cons
ider using monitoring tools to detect and alert on anomalies that might indicate misuse or unexpected behavior.

#### Closing Summary

Given the diverse needs of 50 teams, the optimal approach is to set a default concurrency pattern (e.g., per-key order) that balances complexity and throughput. Teams can then opt up to total order for specific use cases where strict ordering is required. This strategy minimizes the risk of performance degradation while allowing flexibility.

#### Common Mistakes
and Follow-ups
Mistake: Setting total order as the default due to its safety, leading to unnecessary operational overhead.

#### Follow-up
Discussing alternative strategies such as using middleware transformations or implementing custom logic within teams' applications to handle ordering requirements without altering the event bus itself.

**Comparison:**

```
Ordering Guarantee	Throughput	Complexity	Use Case
None	High	Low	Real-time notifications, logs, etc.
Per-key	Moderate	Medium	User-specific actions, updates, etc.
Total Order	Low	High	Financial transactions, critical processes
```

**Code:**

```python
# Example of a simple function to log events with per-key order guarantee
def process_event(key, event):
    key_queue = get_key_queue(key)
    key_queue.put(event)

# Example of managing global state for different keys
key_queues = {}

def initialize_key_queue(key):
    if key not in key_queues:
        key_queues[key] = Queue()
```

**Example input / output:**

```text
Input:
- Team 1 (Notifications) sends an event with key "user123".
- Team 2 (Order Processing) sends an event with key "order456".

Output:
- For Team 1, the event is processed in a non-ordering manner.
- For Team 2, the event is strictly ordered within its own stream of events for "order456".
```

**Data:**

```json
{
  "teams": [
    {
      "id": 1,
      "name": "Notifications",
      "orderingGuarantee": "none"
    },
    {
      "id": 2,
      "name": "Order Processing",
      "orderingGuarantee": "total"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Event
4. Is Key Present?
5. Process Event in Queue
6. Initialize Queue for Key
7. Process Event

