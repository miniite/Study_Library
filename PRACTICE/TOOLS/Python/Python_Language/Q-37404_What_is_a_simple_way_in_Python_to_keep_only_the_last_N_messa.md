---
type: "practice"
difficulty: "Intermediate"
---

# What is a simple way in Python to keep only the last N messages of a chat history so you don't blow the context window on a long conversation?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Token Budgeting for RAG](<../../../../TOOLS/Python/Python_Language/Token_Budgeting_for_RAG.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to implement a function that maintains a rolling buffer of the latest (N) messages in a chat history. The key challenge is ensuring that the implementation efficiently updates and retrieves these messages as new ones are added.

#### Triaging the Question

The question asks for a simple way to keep only the last (N) messages in Python, which means we need to:

Understand how to maintain a limited set of elements.
Handle adding new messages while keeping only the most recent ones.

#### First Principles Solution

Let's break down the problem from first principles:

Data Structure: We need a data structure that can store and manage the last (N) messages efficiently.
Operations:
Adding a new message.
Retrieving the current set of (N) messages.

#### Implementation

For simplicity, we'll use a list to maintain the messages. Python's list is dynamic and allows easy append operations, making it suitable for this task.

PYTHON
def keep_last_n_messages(messages: list[str], n: int) -> list[str]:
    """
    Maintains only the last N messages in a chat history.

    Args:
        messages (list[str]): The current list of messages.
        n (int): The number of latest messages to keep.

    Returns:
        list[str]: A list containing only the last N messages.
    """
    # If we have more than N messages, remove the oldest ones
    if len(messages) > n:
        while len(messages) > n:
            messages.pop(0)

    return messages

# Example usage
messages = ["Hello", "How are you?", "I'm good, thanks!", "That's great.", "What have you been up to?"]
n = 3
last_n_messages = keep_last_n_messages(messages, n)
print(last_n_messages)  # Output: ['I'm good, thanks!', 'That's great.', 'What have you been up to?']

#### Closing Summary
and Common Mistakes

This solution works well for small values of (N). However, there are a few common pitfalls:

Performance: Using a list for this purpose can be inefficient if (N) is large or the list operations are performed frequently.
Memory Usage: Storing all messages can consume significant memory, especially with a large history.

#### Common Mistakes
Using messages = messages[-n:] directly: This approach fails to handle cases where the number of messages is less than (N).
Not managing edge cases properly: If you add a new message and the list already has fewer than (N) elements, it should not remove any.
ASCII Flowchart for Clarity
No
Yes
Add New Message
Is messages length > N?
Return messages
Remove Oldest Messages
Add New Message to End

**Code:**

```python
def keep_last_n_messages(messages: list[str], n: int) -> list[str]:
    if len(messages) > n:
        while len(messages) > n:
            messages.pop(0)

    return messages
```

**Example input / output:**

```text
messages = ["Hello", "How are you?", "I'm good, thanks!", "That's great.", "What have you been up to?"]
n = 3
last_n_messages = keep_last_n_messages(messages, n)
print(last_n_messages)  # Output: ['I\'m good, thanks!', 'That\'s great.', 'What have you been up to?']
```

