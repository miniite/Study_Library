---
type: "practice"
difficulty: "Intermediate"
---

# What is the 'finally' block guaranteed to do, and does it run even if the try block returns or raises?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Python, the finally block is designed to ensure that a specific code segment always executes, regardless of whether an exception is raised in the try block or the function returns normally from the try block. This guarantees that cleanup operations can be performed reliably.

#### Triaging the Question

The question is asking about the behavior and guarantees provided by the finally block in Python. Specifically, it probes your understanding of how exceptions are handled within a try-except-finally structure. We need to consider:

What happens when an exception occurs inside the try block.
What happens if the function returns normally from the try block without raising any exceptions.
The behavior of the finally block in both scenarios.

#### First Principles Solution

The finally block is guaranteed to run after a try block, whether or not an exception was raised. It ensures that certain cleanup operations are performed even if an error occurs or if the function returns normally.

Exception Occurrence:

If an exception is raised within the try block and not caught by any except blocks:
PYTHON
try:
    # Code that might raise an exception
    1 / 0
except ZeroDivisionError as e:
    print(f"Caught {e}")
finally:
    print("Finally block is executed")
The output will be:
CODE
Caught division by zero
Finally block is executed

Normal Return:

If the try block returns without raising any exceptions, the finally block still runs after the return statement.
PYTHON
def divide(a, b):
    try:
        return a / b  # No exception raised here
    finally:
        print("Finally block is executed")

result = divide(10, 2)
print(f"Result: {result}")
The output will be:
CODE
Finally block is executed
Result: 5.0

#### Closing Summary
and Common Mistakes/Follow-Ups

The finally block guarantees that its code executes whether an exception occurs or not, making it ideal for cleanup tasks such as closing files, releasing resources, or logging important information.

#### Common Mistakes

Thinking the finally block will only run if an exception is raised.
Assuming that the try block must contain some potentially error-prone code; the try can also be used to group related statements without necessarily involving exceptions.

#### Follow-Ups

Discussing how the else and finally blocks interact in a more complex try-except-finally-else structure.
Exploring scenarios where multiple except clauses are present with nested try blocks.

#### Cons
idering performance implications when using finally for extensive operations.

**Code:**

```python
# Example of exception handling and finally block
def divide(a, b):
    try:
        return a / b  # No exception raised here
    except ZeroDivisionError as e:
        print(f"Caught {e}")
    finally:
        print("Finally block is executed")

result = divide(10, 2)
print(f"Result: {result}")

# Example with an exception
try:
    1 / 0
except ZeroDivisionError as e:
    print(f"Caught {e}")
finally:
    print("Finally block is executed")
```

**Example input / output:**

```text
Finally block is executed
Result: 5.0

Caught division by zero
Finally block is executed
```

