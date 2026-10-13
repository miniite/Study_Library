---
type: "practice"
difficulty: "Deep"
---

# Your agent occasionally loops, repeating the same tool with the same args; how would you detect the cycle in its action history efficiently, and what state do you hash to declare a loop versus legitimate retries?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently detect cycles in an agent's action history, we need to design a system that can track and identify repeated sequences of actions with their corresponding arguments. The core idea is to hash states in such a way that we can compare them without storing the entire state space explicitly. This approach allows us to recognize loops while minimizing memory overhead.

#### Triage

The question asks for an efficient method to detect cycles in the agent's action history, specifically focusing on identifying whether the same tool or task with identical arguments is repeated. The key here is to balance between detecting cycles and managing resource constraints such as memory usage.

#### First-Principles Solution

State Representation:

We need a way to represent the state of an agent's action in a compact, yet unique manner.
For simplicity, let’s assume each tool invocation is represented by a tuple (tool_name, *args) where *args can be any hashable arguments (e.g., strings, numbers).

Hashing and State Storage:

Use a hash table to store the history of unique states.
Each state should be hashed into a key that uniquely identifies it.
If we encounter a state that has already been seen, we can conclude there is a cycle.

Cycle Detection Algorithm:

Traverse through the action history and compute the hash for each state.
Store these hashes in a set to check for duplicates efficiently (average O(1) time complexity).
Detailed Steps

Define State Hashing Function:

PYTHON
def state_hash(tool_name, *args):
    # Combine tool name with args and hash them together
    return hash((tool_name,) + tuple(args))

Cycle Detection Logic:

As the agent processes each action, compute its state using state_hash.
Check if this state has already been seen by looking it up in a set of previously encountered states.
If the state is found, a cycle has occurred.

#### Example
Implementation:

PYTHON
def detect_cycle(action_history):
    seen_states = set()

    for action in action_history:
        tool_name, *args = action
        current_state_hash = state_hash(tool_name, *args)

        if current_state_hash in seen_states:
            return True  # Cycle detected

        seen_states.add(current_state_hash)

    return False  # No cycle detected

actions = [
    ("toolA", "arg1", "arg2"),
    ("toolB", "arg3", "arg4"),
    ("toolC", "arg5", "arg6"),
    ("toolA", "arg1", "arg2")  # This should trigger a cycle
]

print(detect_cycle(actions))  # Output: True

#### Closing Summary
and Common Mistakes

#### Common Mistakes

Failing to hash the state correctly, leading to incorrect detection or high false positives.
Not considering edge cases where states might change due to side effects (e.g., random seed changes).

#### Follow-Up
Questions:

How would you handle non-deterministic tools?
What if memory is a significant constraint?

**Code:**

```python
def state_hash(tool_name, *args):
    return hash((tool_name,) + tuple(args))

def detect_cycle(action_history):
    seen_states = set()

    for action in action_history:
        tool_name, *args = action
        current_state_hash = state_hash(tool_name, *args)

        if current_state_hash in seen_states:
            return True  # Cycle detected

        seen_states.add(current_state_hash)

    return False  # No cycle detected

actions = [
    ("toolA", "arg1", "arg2"),
    ("toolB", "arg3", "arg4"),
    ("toolC", "arg5", "arg6"),
    ("toolA", "arg1", "arg2")  # This should trigger a cycle
]

print(detect_cycle(actions))  # Output: True
```

**Example input / output:**

```text
actions = [
    ("toolA", "arg1", "arg2"),
    ("toolB", "arg3", "arg4"),
    ("toolC", "arg5", "arg6"),
    ("toolA", "arg1", "arg2")  # This should trigger a cycle
]

print(detect_cycle(actions))  # Output: True
```

