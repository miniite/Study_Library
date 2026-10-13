---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks you to reason about the amortized complexity of a dynamic array-backed stack under a workload of pushes and pops, including the doubling/halving policy and why halving thresholds matter.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Amortized Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Amortized_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the amortized time complexity analysis of a dynamic array-backed stack with a doubling/halving policy, let's break down the problem step-by-step from first principles. We'll start by understanding the core operations and their costs, then move on to analyzing the amortized cost under different scenarios.

TL;DR

The dynamic array-backed stack using a doubling policy (doubling the size when pushing) has an amortized time complexity of (O(1)) per operation due to摊销成本（amortization）。Halving thresholds can be used in certain cases, but they often complicate the analysis without significant benefits for typical push and pop operations.

#### Triaging the Question

The question is essentially asking us to analyze the efficiency of a stack implemented with a dynamic array, focusing on its amortized complexity. The key aspects being probed are:

Understanding basic stack operations: push and pop.
Analyzing memory reallocation costs.
Evaluating the impact of different scaling policies (doubling vs. halving).
Reasoning about worst-case scenarios.

#### Assumptions
We're dealing with a dynamic array implementation that doubles its size when it runs out of space during a push operation.
The initial capacity is 1 element, and we assume the doubling threshold starts at (2^0).

#### First Principles Solution
Basic Operations
Push: Insert an item into the stack. This typically involves checking if there’s enough space in the array; otherwise, it requires allocating a larger array and copying existing elements.
Pop: Remove the top item from the stack. This operation is O(1) as long as we have the necessary indices.
Amortized Cost Analysis

Let's start with the doubling policy:

Initial Setup:

Initially, the array size (A = 1).

Push Operation:

If there’s enough space in the array, push is O(1).
When the array overflows (i.e., becomes full), we reallocate an array of double the size and copy all elements to the new array.
This involves (n) copies for a stack with (n) items.

Amortization:

To find the amortized cost, consider the total number of allocations and their costs over multiple operations.
Let's track the total number of push operations before an allocation is needed: when the array size reaches 1, it triggers a first reallocation; at 2, another; at 4, yet another. This pattern follows powers of two.

Using the Aggregate Accounting Method:

Each push operation costs (O(1)) if there’s space.
Every time we double, the cost is paid by pushing into an array half its current size. Thus, each doubling can be charged to pushes that occur just before it.
Cost Analysis per Doubling:
After 1 push: Cost = 1 (total push operations).
After 2 pushes: Cost = 2 (total push operations), next reallocation.
After 3-4 pushes: Total cost = 4 (3 from initial, 1 for doubling at 2).
This pattern continues with each double costing (n) where (n \approx 2k).

The total number of pushes up to the k-th doubling is approximately (2^k - 1). Hence, the amortized cost per push is: [ \text{Amortized Cost} = O(1) ]

Halving Thresholds

Halving thresholds are rarely used in practice for dynamic arrays due to the following reasons:

Complexity: Introducing a halving policy increases the complexity of the analysis.
Benefit: For typical stack operations (push and pop), the cost is already amortized efficiently with doubling alone.

#### Closing Summary

In summary, a dynamic array-backed stack using a doubling policy has an amortized time complexity of (O(1)) per operation due to the efficient handling of reallocations. Halving thresholds can be considered but are not necessary for optimal performance in typical scenarios involving push and pop operations.

#### Common Mistakes/Follow-Ups
Ignoring Reallocation Costs: Failing to account for the cost of reassigning memory during reallocations.
Misunderstanding Amortization Techniques: Not applying techniques like Aggregate Accounting or Potential Functions correctly.
Overcomplicating Halving Policies: Introducing unnecessary complexity without clear benefit.

**Comparison:**

| Pushes | Array Size | Cost Analysis   |
|--------|------------|-----------------|
| 1      | 2          | 1               |
| 2      | 4          | 3 (2 push + 1)  |
| 3      | 8          | 7 (4 push + 3)  |

**Code:**

```python
def dynamic_array_stack():
    array = [None] * 1  # Initial capacity of 1 element
    size = 0

    def push(item):
        nonlocal size
        if size == len(array):  # If the array is full, reallocate with double the size
            new_array = [None] * (len(array) * 2)
            for i in range(size):
                new_array[i] = array[i]
            array = new_array
        array[size] = item
        size += 1

    def pop():
        nonlocal size
        if size > 0:
            item = array[size - 1]
            size -= 1
            # Halve the array only when necessary, not recommended for push/pop
            if (size < len(array) // 4 and len(array) >= 8):
                new_array = [None] * (len(array) // 2)
                for i in range(size):
                    new_array[i] = array[i]
                array = new_array
            return item
        return None

    # Example usage:
    stack = dynamic_array_stack()
    stack.push(1)
    stack.push(2)
    print(stack.pop())  # Output: 2
```

**Example input / output:**

| Push Input | Stack State      |
|------------|------------------|
| 1          | [1]              |
| 2          | [1, 2]           |
| 3          | [1, 2, 3]        |
| Pop Output | Popped Value     |
|------------|------------------|
|            |                  |

**Data:**

```json
{
  "pushes": [1, 2, 3],
  "pops": [2]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Push
6. Array Full?
7. Allocate Double Size
8. (Copy Elements, 1 Push Cost)
9. Increment Size, Add Item
10. Add Item
11. Check Halving Threshold
12. Halve Array & Copy Elements
13. Update Size, Return LastElement

