---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a simple stack to track nested tool or function calls an LLM agent makes, and why is a stack the right choice?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a stack for tracking nested tool or function calls made by an LLM agent, we need to understand that stacks are ideal data structures due to their Last-In-First-Out (LIFO) nature. This makes them perfect for managing the call stack in a program where functions can be called recursively and need to keep track of which ones were called first.

#### Triaging the Question

The question is essentially asking us to design a mechanism to manage function calls in an LLM agent, ensuring that we can keep track of the sequence in which these calls are made. The interviewer is probing our understanding of data structures and their use cases, particularly focusing on stacks and their properties.

#### Assumptions
Function Calls: Each call will be a distinct function or tool.
Nested Calls: Functions might call other functions, creating a hierarchy of calls.
Error Handling: We need to handle potential stack overflow errors due to excessive nested calls.

#### First-Principles Solution

A stack is the right choice for this problem because:

LIFO Property: It ensures that the most recently called function (or tool) is the first one to be completed and returned from.
Efficiency: Stack operations like push and pop are O(1), making them highly efficient.

Here’s a step-by-step implementation:

Step 1: Define the Stack Structure

We need a stack that can store function call information, such as the name of the function or tool being called.

PYTHON
class CallStack:
    def __init__(self):
        self.stack = []

    def push(self, item):
        self.stack.append(item)

    def pop(self):
        if not self.is_empty():
            return self.stack.pop()
        else:
            raise IndexError("pop from empty stack")

    def peek(self):
        if not self.is_empty():
            return self.stack[-1]
        else:
            raise IndexError("peek from empty stack")

    def is_empty(self):
        return len(self.stack) == 0

    def size(self):
        return len(self.stack)
Step 2: Implement Function Calls Using the Stack

Each time a function or tool calls another, we push its name onto the stack. When it returns, we pop from the stack.

PYTHON
class LLMAgent:
    def __init__(self):
        self.call_stack = CallStack()

    def call_function(self, func_name):
        # Push the current function name to the stack
        self.call_stack.push(func_name)
        print(f"Called: {func_name} (Current stack size: {self.call_stack.size()})")

    def return_from_function(self):
        # Pop from the stack when returning from a function
        func_name = self.call_stack.pop()
        print(f"Returned from: {func_name} (Current stack size: {self.call_stack.size()})")
Step 3: Example Usage

Let's demonstrate with an example where functions A calls B, which in turn calls C.

PYTHON
agent = LLMAgent()

# Function A calls B
def function_A():
    agent.call_function("function_B")

# Function B calls C
def function_B():
    agent.call_function("function_C")
    # Simulate doing some work...
    agent.return_from_function()  # Return from C

# Function C does its work and returns
def function_C():
    # Simulate doing some work...
    agent.return_from_function()

# Start the chain of calls
agent.call_function("function_A")  # Initial call to A

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using a stack is an excellent choice for managing nested function or tool calls due to its efficient LIFO properties. The key implementation involves defining a simple stack class and integrating it into the agent’s logic.

#### Common Mistakes

Forgetting to check if the stack is empty before performing pop operations.
Misunderstanding how to handle stack overflow, which can be mitigated by setting limits or using try-except blocks for safe handling.

#### Follow-Ups

How would you optimize this implementation further?
What are potential issues with deep nesting in function calls and how might you address them?

**Code:**

```python
class CallStack:
    def __init__(self):
        self.stack = []

    def push(self, item):
        self.stack.append(item)

    def pop(self):
        if not self.is_empty():
            return self.stack.pop()
        else:
            raise IndexError("pop from empty stack")

    def peek(self):
        if not self.is_empty():
            return self.stack[-1]
        else:
            raise IndexError("peek from empty stack")

    def is_empty(self):
        return len(self.stack) == 0

    def size(self):
        return len(self.stack)

class LLMAgent:
    def __init__(self):
        self.call_stack = CallStack()

    def call_function(self, func_name):
        # Push the current function name to the stack
        self.call_stack.push(func_name)
        print(f"Called: {func_name} (Current stack size: {self.call_stack.size()})")

    def return_from_function(self):
        # Pop from the stack when returning from a function
        func_name = self.call_stack.pop()
        print(f"Returned from: {func_name} (Current stack size: {self.call_stack.size()})")
```

**Example input / output:**

```text
agent = LLMAgent()

# Function A calls B
def function_A():
    agent.call_function("function_B")

# Function B calls C
def function_B():
    agent.call_function("function_C")
    # Simulate doing some work...
    agent.return_from_function()  # Return from C

# Function C does its work and returns
def function_C():
    # Simulate doing some work...
    agent.return_from_function()

# Start the chain of calls
agent.call_function("function_A")  # Initial call to A
```

