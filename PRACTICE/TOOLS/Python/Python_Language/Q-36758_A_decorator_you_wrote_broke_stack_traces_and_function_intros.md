---
type: "practice"
difficulty: "Intermediate"
---

# A decorator you wrote broke stack traces and function introspection for downstream tooling. What went wrong and how do functools.wraps and signature preservation fix it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, a Python decorator written by an engineer caused issues with stack traces and function introspection, leading to confusion in downstream tooling. This can happen when the decorator modifies the decorated function’s metadata such as its name, docstring, or signature, which are crucial for debugging and code analysis tools.

#### Triage

The question is asking about how to properly handle decorators so that they do not interfere with stack traces and introspection mechanisms in Python. The key points being probed include understanding of the functools.wraps decorator and the importance of preserving function signatures during decoration.

#### First-Principles Solution

Python decorators are used to wrap functions, providing additional functionality without modifying their original code. However, if not implemented correctly, they can break the metadata associated with these functions, leading to issues in debugging and introspection tools.

Problem Analysis

When a decorator modifies the decorated function's metadata (e.g., name or docstring), it can cause problems:

Stack Traces: Tools like pdb rely on accurate stack trace information to point users to the correct line of code.
Introspection: Libraries that inspect functions for documentation, parameters, etc., require unaltered metadata.

#### Solution
Using functools.wraps

To avoid these issues, use the functools.wraps decorator when writing custom decorators. This tool ensures that the decorated function retains its original name, docstring, and other important attributes.

Here’s how to implement a proper decorator using functools.wraps:

PYTHON
from functools import wraps

def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        print("Before calling the function")
        result = f(*args, **kwargs)
        print("After calling the function")
        return result
    return wrapper

@my_decorator
def example_function():
    """This is an example function."""
    print("Inside the example function")

# Running the decorated function

#### example
_function()
Explanation of functools.wraps
wraps: This decorator takes a single argument, which is the original function. It updates the wrapper’s attributes to match those of the wrapped function.
name: The name of the decorated function remains unchanged.
doc: The docstring of the decorated function stays intact.
Signature Preservation

In addition to functools.wraps, preserving the signature is essential. Python's built-in inspect module can be used to get detailed information about functions, including their parameters and return types. By using wraps along with inspect.signature, we ensure that all function details are preserved.

PYTHON
import inspect

def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        print("Before calling the function")
        result = f(*args, **kwargs)
        print("After calling the function")
        return result
    # Preserve the signature of the original function
    try:
        from functools import update_wrapper
        sig = inspect.signature(f)
        update_wrapper(wrapper, f, assigned=inspect._signature)
    except ImportError:
        pass  # Fallback if `update_wrapper` is not available (Python < 3.9)

    return wrapper

@my_decorator
def example_function(x: int) -> str:
    """This is an example function."""
    return "The value of x is " + str(x)

# Check the signature to ensure it's preserved
print(inspect.signature(example_function))  # Should show the correct signature

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to use functools.wraps can lead to misattributed stack traces and incorrect introspection information.

#### Follow-Up
Questions:
How do you handle decorators for asynchronous functions?
Can you modify the example decorator to also preserve type hints?

By following these practices, we ensure that our decorators behave correctly in all scenarios, maintaining compatibility with Python’s built-in tools.

