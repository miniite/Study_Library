---
type: "practice"
difficulty: "Deep"
---

# A batch job processes millions of rows and OOMs near the end even though each row is small. What Python-specific memory pitfalls would you check first?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Memory Leaks in Batch Jobs](<../../../../TOOLS/Python/Python_Language/Memory_Leaks_in_Batch_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a batch job processing millions of rows in Python encounters an Out-of-Memory (OOM) error near the end, despite each row being small on its own, it’s crucial to investigate several Python-specific memory management issues. These include object creation and garbage collection, especially with large data structures or cyclic references.

Let's break down the potential pitfalls step by step:

#### Triage

The key questions are: Why do we suddenly hit OOM? Are objects not being properly managed despite their individual size? How can we ensure that Python’s memory management is efficient?

#### Assumptions
and Context
The rows being processed have a similar structure.
Memory usage increases steadily until it spikes near the end of processing.
Python’s garbage collector (GC) is running, but might not be handling cyclic references effectively.

#### First Principles Solution
Garbage Collection: Ensure that unnecessary objects are properly collected by the GC.
Large Data Structures: Check for large mutable data structures like lists or dictionaries that might be growing unexpectedly.
Cyclic References: Look for cyclic references, which can prevent garbage collection from freeing up memory.
Memory Profiling Tools: Use tools to monitor and profile memory usage in real-time.
Step-by-Step Solution

Check Garbage Collection:

Verify that the GC is running effectively. Python’s gc module provides options to control the behavior of garbage collection.
PYTHON
import gc
# Enable automatic collection if it's disabled
gc.enable()
# Collect manually
gc.collect()

Analyze Large Data Structures:

Monitor the size and growth of large data structures used in the processing loop.
PYTHON
import sys

def track_memory_usage():
    return f"Memory usage: {sys.getsizeof(your_large_structure)}"
If a large structure is growing unexpectedly, investigate why. It might be due to repeated appends or updates.

Detect Cyclic References:

Use the gc module’s get_referents and is_referring_objects methods to identify cyclic references.
PYTHON
import gc

def find_cyclic_references(obj):
    return [o for o in gc.get_referents(*[obj]) if gc.is_referring_objects(o, obj)]

Memory Profiling:

Use third-party libraries like memory-profiler or tracemalloc to monitor memory usage.
PYTHON
from memory_profiler import profile

@profile
def process_data():
    # Your data processing logic here
    pass

Optimize Data Structures:

#### Cons
ider using more efficient data structures like array.array or NumPy arrays for large datasets.
PYTHON
import numpy as np

arr = np.zeros((10000, 100))

Parallel Processing and Threading:

If possible, split the processing into smaller chunks or use parallel processing to distribute memory usage more evenly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically checking these areas, you can identify the root cause of the OOM errors and optimize your Python code. Common mistakes include not using garbage collection effectively, overlooking cyclic references, and not properly managing large data structures.

#### Follow-ups

How would you integrate these checks into a production environment?
Are there any specific Python libraries or tools that could help with memory management in this scenario?

**Code:**

```python
import gc
import sys

def track_memory_usage():
    return f"Memory usage: {sys.getsizeof(your_large_structure)}"

def find_cyclic_references(obj):
    return [o for o in gc.get_referents(*[obj]) if gc.is_referring_objects(o, obj)]
```

