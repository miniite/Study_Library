---
type: "practice"
difficulty: "Deep"
---

# How does the contextlib.contextmanager decorator turn a generator into a context manager?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Generators and Streaming ETL](<../../../../TOOLS/Python/Python_Language/Generators_and_Streaming_ETL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how contextlib.contextmanager turns a generator function into a context manager, let's first break down what these concepts are and then dive into the details of this decorator.

TL;DR:

The @contextlib.contextmanager decorator transforms a generator function into a context manager by providing two methods: __enter__ and __exit__. The generator function yields control back to its caller in the __enter__ method and resumes execution when the __exit__ method is called.

#### Triaging the Question

The core of this question involves understanding Python's context management protocol, which includes the methods __enter__ and __exit__. The contextlib.contextmanager decorator simplifies creating such protocols. We need to understand how a generator function can be made to behave as if it has these two methods defined.

#### First-Principles Solution
Understand Context Managers: A context manager is an object that defines the runtime context to with statements. It must have __enter__ and __exit__ methods.
Generator Function: Generators in Python are a type of iterable, like lists or tuples. However, they do not need to be fully realized upon creation; instead, their computation is deferred until the generator's next() method is called.
The contextlib.contextmanager Decorator:
This decorator wraps a function that yields control back to its caller and then resumes execution when __exit__ is called.
It automatically provides the __enter__ and __exit__ methods for you.

Here’s how it works in detail:

#### Example
Code

Let's take an example of using @contextlib.contextmanager:

PYTHON
import contextlib

@contextlib.contextmanager
def simple_context_manager():
    print("Entering the block")
    try:
        yield "resource"
    finally:
        print("Exiting the block")

# Usage
with simple_context_manager() as resource:
    print(f"Resource: {resource}")
Step-by-Step Explanation:

Defining the Context Manager:

The simple_context_manager function is a generator.
It uses yield to control when execution should resume.

Yielding Control:

When you use with simple_context_manager() as resource, Python calls the __enter__ method of the context manager, which in this case just returns "resource".
The value returned by yield is assigned to the variable after the as.

Executing the Block:

The code within the with block runs normally.
In our example, it prints "Resource: resource".

Resuming Execution on Exit:

When exiting the with block (either normally or due to an exception), Python calls the __exit__ method of the context manager.
The finally block in the generator ensures that this is always called.

Automatic Context Management:

@contextlib.contextmanager wraps the function and provides __enter__ and __exit__ methods for free, making it easy to implement complex resource management logic.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Common Mistake: Forgetting to handle exceptions in the generator. If an exception is raised within the context manager’s block, it should be properly handled by the __exit__ method.

#### Follow-Up
Discussing scenarios where you might want to use a custom context manager (e.g., for database transactions or file handling).
Further Reading: Explore advanced topics like nested context managers and how they interact with each other.

This explanation covers the fundamental idea of using generators with @contextlib.contextmanager and provides insights into its practical applications. Understanding these concepts is crucial for effective resource management in Python.

