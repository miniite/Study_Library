---
type: "practice"
difficulty: "Intermediate"
---

# Your agent occasionally loops forever calling the same tool with the same args. How would you detect and break the loop in Python, and what state would you track across turns?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of an AI agent looping infinitely by repeatedly calling the same function or tool with identical arguments, we need to implement a mechanism that tracks the history of calls made and ensures the loop is broken when necessary. This can be achieved in Python by using a state management approach.

The key idea here is to maintain a record of previously executed actions so that if an action is repeated, the agent can recognize it and take appropriate corrective measures, such as breaking out of the loop or taking alternative actions.

#### Triaging the Question

The core of the question revolves around detecting and handling infinite loops in Python. The interviewer is probing for your understanding of state management, hash tables, and conditional logic in Python. We will assume that the agent's actions are represented by function calls, each with specific arguments, and we need to ensure these do not repeat indefinitely.

#### First-Principles Solution
Define a State Tracker: A dictionary can be used to track previous states or function calls.
Capture Function Arguments: Each call’s arguments will be hashed to uniquely identify it.
Check for Repeats: Before making each new call, check if the argument hash already exists in the state tracker.
Implement a Break Mechanism: If a repeat is detected, break out of the loop or take an alternative action.

#### Implementation
Steps
Define a function to handle the agent's task with a mechanism for detecting repeats.
Use hashlib to create unique identifiers (hashes) for each set of arguments.
Maintain a state dictionary in memory that tracks these hashes.

Here’s how you can implement this in Python:

PYTHON
import hashlib

def detect_and_break_loop():
    # State tracker to store previously seen function calls
    seen_hashes = set()

    def process_action(action, args):
        # Hash the arguments to create a unique identifier
        args_str = str(args)
        hash_value = hashlib.md5(args_str.encode()).hexdigest()

        if hash_value in seen_hashes:
            print("Loop detected! Breaking out of the loop.")
            return False  # Indicate loop break
        else:
            seen_hashes.add(hash_value)
            print(f"Processing action: {action} with args: {args}")
            # Simulate processing logic here
            return True

    # Example usage:
    process_action('analyze_data', (1, 2, 3))
    process_action('analyze_data', (4, 5, 6))
    process_action('analyze_data', (1, 2, 3))  # This should detect the loop

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we have successfully tracked function calls using a hash set to prevent infinite loops. The key is maintaining state across turns in a way that allows us to recognize repeating patterns.

#### Common Mistakes

Not hashing arguments properly or losing their structure.
Failing to initialize the state tracker before starting actions.
Overcomplicating the logic with unnecessary features.

For follow-ups, you could discuss how this approach scales for complex systems, consider performance implications of hash table lookups, and explore alternatives like using a database to store states across multiple agents or sessions.

**Code:**

```python
import hashlib

def detect_and_break_loop():
    seen_hashes = set()

    def process_action(action, args):
        args_str = str(args)
        hash_value = hashlib.md5(args_str.encode()).hexdigest()

        if hash_value in seen_hashes:
            print("Loop detected! Breaking out of the loop.")
            return False
        else:
            seen_hashes.add(hash_value)
            print(f"Processing action: {action} with args: {args}")
            return True

    process_action('analyze_data', (1, 2, 3))
    process_action('analyze_data', (4, 5, 6))
    process_action('analyze_data', (1, 2, 3))  # This should detect the loop
```

**Flow:**

1. Yes
2. No
3. Start
4. Is action repeated?
5. Break loop
6. Process action and record it
7. End

