---
type: "practice"
difficulty: "Deep"
---

# As architect you must set the org's stance on shared mutable state: ban it (immutability + message passing) or manage it (disciplined locking). Argue the long-run defect-rate and hiring/onboarding implications of each stance across a fast-growing engineering org.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In setting our organization's stance on shared mutable state, we're essentially deciding between a ban on shared mutable state (favoring immutability and message passing) or managing it carefully through disciplined locking mechanisms. This decision has profound implications for the long-run defect rate, as well as hiring and onboarding processes in a fast-growing engineering organization.

#### Triaging the Question

The core of this question is to evaluate which approach—immutability and message-passing (functional programming paradigm) or disciplined locking (object-oriented concurrency)—is better suited for a rapidly expanding development team. We need to consider how these paradigms affect code quality, maintainability, developer productivity, and overall software reliability.

#### Assumptions
The organization is growing quickly with multiple teams working on different services.
Developers are familiar with both paradigms but might lean towards one over the other.
The goal is to minimize defects while ensuring scalable and maintainable codebases.

#### First Principles Solution
Immutability and Message Passing (Ban Shared Mutable State)
Code Quality: Functional programming, which emphasizes immutability, can lead to more predictable and testable code. It's easier to reason about the state of a system when it doesn't change unexpectedly.
Maintainability: Easier to debug issues as side effects are reduced or eliminated entirely. Code is often more modular and reusable.
Defect Rate: Lower defect rate due to fewer race conditions and simpler error handling mechanisms. This can be quantified by tracking the number of bugs introduced per code commit.
Hiring/Onboarding: Easier onboarding process as developers don't need extensive training in concurrency and threading. They can focus more on functional patterns, which are generally easier to learn.
Disciplined Locking (Manage Shared Mutable State)
Code Quality: Properly managed shared mutable state can lead to efficient multi-threaded applications but requires a deep understanding of synchronization primitives.
Maintainability: Can be harder to maintain as the codebase grows, especially in distributed systems where race conditions are more complex and harder to predict.
Defect Rate: Higher defect rate due to potential issues with deadlocks, data races, and improper locking mechanisms. Debugging such issues can be time-consuming.
Hiring/Onboarding: More challenging onboarding process as developers need to understand concurrency patterns deeply. This can lead to higher training costs and longer ramp-up times.

#### Closing Summary

Given the fast-paced growth of our engineering organization, adopting a functional programming paradigm with immutability and message passing is likely more beneficial in the long run. It reduces defects, improves code quality, and simplifies onboarding processes. However, it’s important to note that disciplined locking can be more appropriate for specific use cases where performance optimizations are critical.

#### Common Mistakes/Follow-Ups
Ignoring Real-world Trade-offs: While immutability is easier, it might not always be the most efficient solution in all scenarios.
Overlooking Learning Curves: The initial learning curve for functional programming can be steep but is manageable with proper training and resources.
Failing to Adapt: Organizations should be flexible enough to adapt their stance as team skills evolve and project requirements change.

**Code:**

```python
# Example of a simple immutable function in Python
def add(a, b):
    return a + b

result = add(3, 4)  # result is 7, no shared mutable state
```

**Example input / output:**

```
Input	Output
add(3, 4)	7
Diagrams
does not share mutablestate
Immutable Function
result = 7
No Race Conditions
```

**Data:**

```json
{
  "users": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ]
}
```

