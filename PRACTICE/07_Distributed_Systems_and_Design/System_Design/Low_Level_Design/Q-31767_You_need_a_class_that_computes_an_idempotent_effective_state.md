---
type: "practice"
difficulty: "Expert"
---

# You need a class that computes an idempotent 'effective state' from a stream of possibly-reordered updates using timestamps/version vectors; how do you decide last-writer-wins vs merge and where does clock skew bite?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a class that computes an idempotent 'effective state' from a stream of possibly-reordered updates using timestamps or version vectors, we must decide between last-writer-wins (LWW) and merge strategies based on the requirements of our system. Clock skew can introduce subtle but significant issues with both approaches. This decision involves understanding the trade-offs in consistency, scalability, and fault tolerance.

#### Triaging the Question

The core question is about choosing an appropriate strategy to resolve conflicting updates in a distributed system where updates may arrive out of order or with skewed clocks. We need to decide whether last-writer-wins (LWW) or merge operations are better suited for our use case. The decision will impact how we handle clock skew and ensure eventual consistency.

Key Assumptions
Timestamps/Version Vectors: These identify the source and sequence of updates.
Clock Skew: Differences in wall-clock times across nodes can lead to conflicts.
Conflict Resolution Requirements: We need to maintain a single, consistent view of state despite potential reordering or skew.

#### First-Principles Solution

1. Last-Writer-Wins (LWW) Strategy:

Simplicity: Easy to implement and understand.

#### Cons
istency Issues: Clock skew can cause writes from earlier in time to overwrite later ones, leading to potential data loss or inconsistency.

2. Merge Strategy:

Complexity: More complex logic is required to merge updates without losing information.
Fault Tolerance: Better at handling clock skew and maintaining consistency over time by combining multiple changes.
Choosing Between LWW and Merge
Simplicity vs. Fault Tolerance: For systems with low risk of clock skew, LWW can be simpler but may introduce data loss. For high-risk environments, merge provides a more robust solution.

#### Performance Considerations
Merging operations are often more compute-intensive than LWW.

#### Example Scenario

#### Cons
ider an e-commerce application where products have multiple attributes that can change over time (e.g., price, stock). If two updates arrive simultaneously with conflicting changes, we need to decide how to reconcile them.

LWW:

If node A and node B both update the price of a product at nearly the same timestamp but slightly different times, LWW would result in only one update being applied, potentially losing valuable information.

Merge:

Merging these updates could involve averaging prices or finding a compromise value to ensure no data is lost.

#### Closing Summary

In deciding between last-writer-wins and merge strategies for handling possibly-reordered updates, we must consider the system's requirements, clock skew risk, and the complexity of implementation. LWW offers simplicity but risks losing important information due to clock skew, while merge strategies provide better fault tolerance at the cost of added complexity.

#### Common Mistakes/Follow-Ups
Overlooking Clock Skew: Many systems assume a perfectly synchronized clock, which is unrealistic in distributed environments.
Ignoring Data Loss: LWW can lead to data loss if updates with slightly different timestamps are applied based on clock skew.
Underestimating Complexity: Merge operations require careful implementation and testing to ensure consistency across all possible update scenarios.

For follow-ups, we might explore specific algorithms for merge resolution or discuss how to handle different types of conflicts in a distributed system.

**Comparison:**

| Strategy | Simplicity | Fault Tolerance | Example Conflict |
|----------|------------|-----------------|------------------|
| LWW      | High       | Low             | Last update wins |
| Merge    | Low        | High            | Merged value     |

**Code:**

```
Example Python class implementing LWW strategy

class IdempotentStateCalculator: def init(self): self.state = {}

CODE
Copy
def update(self, key, value, timestamp):
    if key not in self.state or self.state[key]['timestamp'] < timestamp:
        self.state[key] = {'value': value, 'timestamp': timestamp}

def get_effective_state(self):
    return {k: v['value'] for k, v in self.state.items()}
CODE
Copy

```python
# Example Python class implementing merge strategy
class MergeStateCalculator:
    def __init__(self):
        self.state = {}

    def update(self, key, value, timestamp):
        if key not in self.state or self.merge(value, self.state[key]['value'], timestamp > self.state[key]['timestamp']):
            self.state[key] = {'value': value, 'timestamp': timestamp}

    def merge(self, a, b, newer_a):
        # Simple example: average prices
        return (a + b) / 2 if newer_a else a

    def get_effective_state(self):
        return {k: v['value'] for k, v in self.state.items()}
```

**Example input / output:**

```text
# Example input and output for LWW strategy
calculator = IdempotentStateCalculator()
calculator.update('product1', 100, 1632456789)
calculator.update('product2', 150, 1632456790)
calculator.get_effective_state()  # Output: {'product1': 100, 'product2': 150}

# Example input and output for merge strategy
calculator = MergeStateCalculator()
calculator.update('product1', 100, 1632456789)
calculator.update('product2', 150, 1632456790)
calculator.get_effective_state()  # Output: {'product1': 125.0, 'product2': 150}
```

**Data:**

```json
{
  "product1": {
    "price": 100,
    "timestamp": 1632456789
  },
  "product2": {
    "price": 150,
    "timestamp": 1632456790
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Update
4. Is Clock Skew Critical?
5. Merge Strategy
6. LWW Strategy
7. Apply Merge Logic
8. Last Writer Wins
9. Output Merged State
10. Output Last Write State

