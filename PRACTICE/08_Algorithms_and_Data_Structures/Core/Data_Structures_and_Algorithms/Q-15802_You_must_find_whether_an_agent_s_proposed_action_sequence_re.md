---
type: "practice"
difficulty: "Deep"
---

# You must find whether an agent's proposed action sequence revisits a previously explored world-state to prune redundant planning; what hashing of state gives you O(1) visited checks, and how do you handle near-equivalent states?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

To determine if an agent’s proposed action sequence revisits a previously explored world-state, we can use hashing to achieve constant-time (O(1)) checks on visited states. However, handling near-equivalent states requires careful consideration of state representation and equivalence checking mechanisms.

#### Triage

The core problem is to efficiently detect revisited states in a planning context using data structures and algorithms. We will explore how to design an efficient hash-based solution while acknowledging the challenge of near-equivalent states.

#### First-Principles Solution

State Representation:

The first step is defining what constitutes a state. In this case, each world-state can be represented as a tuple or a dictionary containing relevant variables such as position, object configurations, and environmental conditions.

Hash Function Design:

We need to design a hash function that maps each state to a unique identifier (hash). For complex states like those in an environment with multiple objects, we can use a combination of hashing techniques:
PYTHON
def state_hash(state):
    # Example for simple tuple state
    return hash(tuple(sorted(state.items())))

    # Example for object configuration state
    combined_state = ''.join([str(v) for v in state.values()])
    return hash(combined_state)

Hash Table Implementation:

Use a hash table (dictionary or set) to store and look up states.
PYTHON
  visited_states = {}

  def add_state(state):
      hashed_state = state_hash(state)
      if hashed_state not in visited_states:
          visited_states[hashed_state] = True

  def check_state(state):
      hashed_state = state_hash(state)
      return hashed_state in visited_states

Handling Near-Equivalent States:

Near-equivalent states can be tricky because they might have minor differences that do not affect the overall plan but need to be accounted for.
One approach is to use a tolerance level or fuzzy hashing:
PYTHON
def fuzzy_hash(state):
    # Implement some form of fuzzy matching logic here, e.g., Levenshtein distance
    return hash(tuple(fuzzy_state_representation))
Alternatively, use a more advanced technique like fingerprinting based on the state's properties and relationships.

Scalability Considerations:

The choice of hash function and representation can significantly impact performance. For very large state spaces, consider using techniques like:
Hierarchical hashing
Minwise hashing

#### Closing Summary
and Common Mistakes/Follow-Ups:
Mistake 1: Using a non-uniform or poor hash function that leads to many collisions.
Mistake 2: Not considering the equivalence of states, leading to unnecessary reprocessing.

#### Follow-Up
Questions:
How would you handle very large state spaces?
What are potential false positives and how can they be mitigated?
Depth + Language Match:

For a mid-level engineer, it's crucial to understand both the theoretical underpinnings and practical considerations. For higher levels, we need to consider scalability, performance trade-offs, and more sophisticated techniques.

**Code:**

```python
def state_hash(state):
    return hash(tuple(sorted(state.items())))

visited_states = {}

def add_state(state):
    hashed_state = state_hash(state)
    if hashed_state not in visited_states:
        visited_states[hashed_state] = True

def check_state(state):
    hashed_state = state_hash(state)
    return hashed_state in visited_states
```

**Example input / output:**

```text
# Example input
state1 = {"position": (0, 0), "objects": ["rock", "tree"]}
state2 = {"position": (0, 0), "objects": ["tree", "rock"]}

add_state(state1)
print(check_state(state2))  # Output: True
```

**Flow:**

1. State Representation
2. Hash Function Design
3. Hash Table Implementation
4. Handling Near-EquivalentStates
5. Scalability Considerations

