---
type: "practice"
difficulty: "Expert"
---

# Explain how Python's reference counting plus generational GC can cause latency spikes, and how you'd tune or disable gc for a latency-sensitive worker.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Python Tail Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why Python's garbage collection (GC) mechanism can cause latency spikes in a latency-sensitive application, let's first break down what reference counting and generational GC are and how they interact. Then we'll discuss the scenarios where these mechanisms might introduce performance issues and how to mitigate them.

TL;DR

Python's GC consists of reference counting along with generational collection, which can lead to unpredictable pauses during garbage collection. To address this in latency-sensitive applications, you can tune or disable the GC mechanism or use a more predictable garbage collector like PyPy.

#### Triaging the Question

The question is probing your understanding of Python's memory management and how it affects application performance, specifically in scenarios where low-latency operations are critical. Key assumptions include:

The environment supports a latency-sensitive workload.
You have some control over the GC settings or can choose an alternative implementation.

#### First Principles Solution
Reference Counting

Python uses reference counting to manage memory. Every object has a reference count, which increments when a new reference is created and decrements when references are removed. When the reference count reaches zero, the object's memory is freed. This mechanism is fast but does not handle cyclic references.

Generational Collection

To address cycles, Python employs generational garbage collection (GGC). GGC categorizes objects into three generations: young, old, and dead. The idea is to optimize the collection process by focusing on younger objects where most of them are likely to be short-lived.

When an object's reference count drops to zero but it still has references in older generations, Python performs a full GC cycle that can pause execution for arbitrary durations, leading to potential latency spikes.

Latency Sensitivity and GC Impact

In latency-sensitive applications, such as real-time services or high-frequency trading systems, these pauses can be catastrophic. The GGC mechanism is designed to minimize the impact on performance, but it still has unpredictable behavior, especially during the major collections of older generations.

Tuning or Disabling GC
Tuning GGC

You can fine-tune Python's GGC by adjusting the gc module settings. For example:

PYTHON
import gc
# Enable automatic collection if not already enabled
gc.enable()
# Set thresholds for minor and major collections
gc.set_threshold(700, 10, 5)

However, these adjustments are generally limited in their effectiveness and might still introduce pauses.

Disabling GC

Disabling GC entirely can be a more radical approach. However, it's important to note that this will lead to memory leaks if not managed properly:

PYTHON
import gc
# Disable automatic collection
gc.disable()

In practice, disabling the GC is rarely recommended due to the risk of memory leaks.

Using PyPy as an Alternative

For applications with stringent latency requirements, using a different Python implementation like PyPy can be a better solution. PyPy uses a more predictable garbage collector that avoids full pauses and provides deterministic behavior:

BASH
pip install pypy
# Run your application under PyPy
pypy my_application.py

#### Closing Summary
+ Common Mistakes/Follow-Ups
Best Practices
Monitor Memory Usage: Regularly monitor the memory usage of your application to detect leaks.
Profiling: Use profiling tools like tracemalloc or memory_profiler to understand where memory is being held.
Code Review: Ensure that all objects are properly managed and cleaned up.

#### Follow-Ups

The interviewer might ask about specific use cases, such as:

How would you implement a custom GC in Python?
What are the trade-offs between reference counting and generational GC?
Can you explain the implications of disabling GC on memory management?
Self-Check

This answer covers both the theoretical aspects and practical solutions for tuning or disabling garbage collection in Python, ensuring correctness and completeness.

**Code:**

```python
import gc

# Example of enabling automatic collection with thresholds set
gc.enable()
gc.set_threshold(700, 10, 5)

# Disabling automatic collection
gc.disable()
```

