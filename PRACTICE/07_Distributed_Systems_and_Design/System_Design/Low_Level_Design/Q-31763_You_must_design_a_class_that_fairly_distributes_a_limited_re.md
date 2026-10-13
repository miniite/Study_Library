---
type: "practice"
difficulty: "Deep"
---

# You must design a class that fairly distributes a limited resource (say 100 concurrent export slots) across tenants so no tenant hogs; compare weighted fair queuing vs simple round-robin at the class level.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of distributing limited resources fairly among multiple tenants, we need to design a class that implements either weighted fair queuing (WFQ) or simple round-robin (RR). The goal is to ensure no tenant monopolizes the resource while still prioritizing certain tenants if needed. Let's explore both approaches in detail.

#### Triaging the Question

The core of this question revolves around understanding and implementing resource allocation strategies that balance fairness with efficiency. Specifically, we need to compare two mechanisms: weighted fair queuing (WFQ) and simple round-robin (RR). The class will manage 100 concurrent export slots for multiple tenants.

#### Assumptions
Each tenant has a unique identifier.
Tenants may have different weights based on their importance or historical usage.
We need to ensure that no single tenant uses all the available slots, maintaining fairness among all tenants.

#### First Principles Solution
Simple Round-Robin (RR)
Initialization: Create a queue for each tenant and maintain an index to track which tenant gets the next slot.
Slot Allocation: Distribute slots in a cyclic manner across tenants. When a tenant uses a slot, increment the index modulo the total number of tenants.
PYTHON
class RoundRobinScheduler:
    def __init__(self, num_tenants):
        self.tenants = [Queue() for _ in range(num_tenants)]
        self.index = 0

    def allocate_slot(self):
        tenant_queue = self.tenants[self.index % len(self.tenants)]
        if not tenant_queue.is_empty():
            slot = tenant_queue.dequeue()
            return slot
        else:
            # Handle the case when no slots are available for this tenant
            pass

# Example usage
rr_scheduler = RoundRobinScheduler(10)
Weighted Fair Queuing (WFQ)
Initialization: Create a priority queue where each tenant is assigned a weight and their requests are prioritized based on these weights.
Slot Allocation: Distribute slots to tenants in proportion to their weights, ensuring that high-weight tenants get more slots but not monopolize them.
PYTHON
import heapq

class WeightedFairQueuingScheduler:
    def __init__(self, num_tenants):
        self.tenants = [(100 // num_tenants, Queue()) for _ in range(num_tenants)]
        self.heap = []  # Store (weight, tenant_id, slot_request)
        self.index = 0

    def allocate_slot(self, tenant_id):
        weight, tenant_queue = self.tenants[tenant_id]
        if not tenant_queue.is_empty():
            slot = tenant_queue.dequeue()
            heapq.heappush(self.heap, (-weight, tenant_id, slot))
            return slot
        else:
            # Handle the case when no slots are available for this tenant
            pass

    def next_slot(self):
        while self.heap:
            weight, tenant_id, slot = heapq.heappop(self.heap)
            if not self.tenants[tenant_id][1].is_empty():
                self.tenants[tenant_id][1].enqueue(slot)
                return slot
        else:
            # No slots available for any tenant
            pass

# Example usage
wfq_scheduler = WeightedFairQueuingScheduler(10)

#### Closing Summary
and Common Mistakes/Follow-Ups

Both WFQ and RR ensure that no single tenant uses all the slots, but they differ in how resources are allocated. RR is simpler to implement and ensures fairness by distributing resources cyclically. However, it does not account for varying importance or historical usage of different tenants.

WFQ, on the other hand, provides a more nuanced approach where high-weight tenants can get more slots while still being constrained so that no single tenant uses all available resources. This approach is particularly useful in scenarios where some tenants need to be prioritized but fairness must still be maintained.

#### Common Mistakes
Misunderstanding the distribution logic: For RR, forgetting to handle the case when a tenant has exhausted its requests. For WFQ, incorrectly managing the priority queue or ignoring weight constraints.
Overloading one tenant: In both approaches, failing to ensure that no single tenant monopolizes all resources.

#### Follow-Ups
What happens if a tenant’s weight changes dynamically?
How do you handle real-time adjustments to resource allocation?
Can we introduce additional constraints or policies in either approach?

**Comparison:**

| Tenant ID | Weight |
|-----------|--------|
| 1         | 10     |
| 2         | 20     |
| 3         | 15     |
| 4         | 5      |
| 5         | 10     |

**Code:**

```python
class Queue:
    def __init__(self):
        self.items = []

    def is_empty(self):
        return len(self.items) == 0

    def enqueue(self, item):
        self.items.append(item)

    def dequeue(self):
        if not self.is_empty():
            return self.items.pop(0)
        else:
            raise IndexError("Queue is empty")
```

**Example input / output:**

```text
RoundRobinScheduler:
Input: num_tenants = 5
Output: Tenants get slots in a round-robin manner.

WeightedFairQueuingScheduler:
Input: num_tenants = 5, weights = [10, 20, 15, 5, 10]
Output: Tenants with higher weights get more slots, but the distribution remains fair.
```

**Data:**

```yaml
tenants: [1, 2, 3, 4, 5]
weights: [10, 20, 15, 5, 10]
slots: 100
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initialize Tenants
6. Tenant Queue Full?
7. Handle No Slots
8. Enqueue Slot
9. Next Slot Request
10. Heap Empty?
11. No Slots Available
12. Dequeue Next Slot

