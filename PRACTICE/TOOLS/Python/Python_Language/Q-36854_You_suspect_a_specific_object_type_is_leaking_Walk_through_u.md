---
type: "practice"
difficulty: "Deep"
---

# You suspect a specific object type is leaking. Walk through using tracemalloc, gc.get_objects, and objgraph to find what's retaining it in a running service.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Memory Leaks in Batch Jobs](<../../../../TOOLS/Python/Python_Language/Memory_Leaks_in_Batch_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address an object leak issue in your Python application, we can leverage several tools including tracemalloc, gc.get_objects(), and objgraph. The goal is to identify which parts of your code or external libraries are retaining references to objects that should be garbage collected. Let's dive into how these tools work together.

Firstly, tracemalloc helps us monitor memory allocations over time. It captures a snapshot at regular intervals, allowing us to see where memory is being allocated and retained. Secondly, gc.get_objects() provides a way to manually inspect the contents of the garbage collector's reachability graph. Lastly, objgraph offers visualization tools that can help us identify object retention issues.

To begin our investigation, we'll start by setting up these tools in a running service.

