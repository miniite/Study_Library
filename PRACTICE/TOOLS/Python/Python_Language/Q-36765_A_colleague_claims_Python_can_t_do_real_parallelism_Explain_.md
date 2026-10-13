---
type: "practice"
difficulty: "Expert"
---

# A colleague claims 'Python can't do real parallelism'. Explain nuances with the GIL, C extensions releasing it, subinterpreters, and free-threaded (3.13) builds.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ The GIL and CPU-bound Work](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/The_GIL_and_CPU-bound_Work.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: While Python's Global Interpreter Lock (GIL) can make true parallelism challenging within a single process, there are several ways to overcome this limitation using C extensions, subinterpreters, and free-threaded builds. These techniques enable real parallelism and concurrency.

The claim that "Python can't do real parallelism" is somewhat misleading. Let's explore the nuances:

Global Interpreter Lock (GIL): The GIL is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once in a single process. This ensures memory management and synchronization are thread-safe but can limit CPU-bound tasks' performance.

C Extensions: By writing C extensions, you can release the GIL temporarily during computationally intensive operations. Libraries like NumPy and Pandas leverage this to achieve parallelism within threads. The key is careful use of with threading.lock() or similar constructs to manage critical sections.

Subinterpreters: Python 3 introduced subinterpreters, allowing multiple isolated execution contexts in a single process. This feature can be used for true parallelism by running different threads with their own interpreter state. For example:

PYTHON
import disnake

def run_subinterp(code):
    interp = disnake.get_event_loop()
    return interp.run_until_complete(interp.execute_code(code))

# Example usage in a thread
threading.Thread(target=run_subinterp, args=("print('Hello from subinterp')",)).start()

Free-Threaded (3.13) Builds: Python 3.13 introduced free-threaded builds, which eliminate the GIL entirely. This allows multiple threads to execute Python code concurrently without any locking. However, this is a complex build configuration and might not be available in standard distributions.

Let's consider an example where we use C extensions and subinterpreters:

PYTHON
import threading

def compute intensive_task(i):
    # Example: Using NumPy for matrix multiplication (C extension)
    import numpy as np
    A = np.random.rand(100, 100)
    B = np.random.rand(100, 100)
    return np.dot(A, B)

def run_subinterp(code):
    interp = disnake.get_event_loop()
    return interp.run_until_complete(interp.execute_code(code))

# Example usage
threads = []
for i in range(4):
    t = threading.Thread(target=compute, args=(i,))
    threads.append(t)
    t.start()

subinterp_thread = threading.Thread(target=lambda: run_subinterp("print('Hello from subinterp')"))
subinterp_thread.start()

In summary, while Python's GIL can limit parallelism in a single process, C extensions, subinterpreters, and free-threaded builds provide ways to achieve true concurrency. Understanding these mechanisms is crucial for optimizing performance-critical applications.

**Code:**

```python
import threading

def compute(intensive_task):
    import numpy as np
    A = np.random.rand(100, 100)
    B = np.random.rand(100, 100)
    return np.dot(A, B)

def run_subinterp(code):
    interp = disnake.get_event_loop()
    return interp.run_until_complete(interp.execute_code(code))

# Example usage
threads = []
for i in range(4):
    t = threading.Thread(target=compute, args=(i,))
    threads.append(t)
    t.start()

subinterp_thread = threading.Thread(target=lambda: run_subinterp("print('Hello from subinterp')"))
subinterp_thread.start()
```

