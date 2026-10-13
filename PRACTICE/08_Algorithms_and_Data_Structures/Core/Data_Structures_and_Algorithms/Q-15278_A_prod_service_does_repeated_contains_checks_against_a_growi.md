---
type: "practice"
difficulty: "Intermediate"
---

# A prod service does repeated 'contains' checks against a growing list that was never converted to a set; how would you catch this class of accidental O(n^2) in code review or profiling?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of repeated contains checks leading to an accidental (O(n^2)) complexity, we need a systematic approach that combines both static analysis and runtime monitoring. During code review, we can identify potential inefficiencies by analyzing the logic. For profiling, we can instrument the application to detect performance bottlenecks.

#### Triaging the Question

The question revolves around identifying and mitigating an (O(n^2)) complexity issue due to repeated contains checks on a growing list in code review and profiling scenarios. The core of the problem is recognizing when such operations are being performed inefficiently, especially if the underlying data structure (list) isn't optimized for membership checks.

Code Review
Identifying Inefficiencies

During code review, we need to look for patterns where contains or similar methods are used repeatedly within nested loops. For example:

PYTHON
for i in range(len(lst)):
    for j in range(len(lst)):
        if lst[j] == target:
            # Do something

This pattern is a red flag as it leads to (O(n^2)) complexity.

Checking Data Structures

Ensure that the underlying data structure used for membership checks supports efficient lookups. A set or hash set would be more appropriate than a list in most cases, but we need to verify if this change was made.

Profiling

To identify performance issues during runtime, we can use profiling tools to measure the execution time of critical sections of code. Here’s a step-by-step approach:

Identify Hotspots: Use a profiler like cProfile in Python or similar tools in other languages to pinpoint where most of the execution time is spent.
Analyze Time Distribution:
PYTHON
import cProfile

def check_performance():
    lst = [i for i in range(10000)]
    target = 5000
    # Simulate repeated contains checks
    for _ in range(100):
        for j in lst:
            if j == target:
                pass

cProfile.run('check_performance()')
Monitor Memory and CPU Usage: Tools like gprof (for C/C++), jvisualvm (Java), or tracemalloc (Python) can help visualize memory usage and pinpoint slow parts of the code.

#### First-Principles Solution

To solve this problem, we can implement a hybrid approach combining static analysis tools with runtime monitoring:

Static Analysis Tool Integration: Integrate static analysis tools like PyLint, ESLint, or Clang-Tidy that can flag potential (O(n^2)) complexity issues.
Dynamic Profiling: Use profiling libraries and frameworks to instrument the application at key points where repeated contains checks are likely to occur.

#### Closing Summary

In summary, catching accidental (O(n^2)) complexity due to repeated contains checks requires a combination of static analysis during code review and dynamic runtime monitoring. Tools like profilers can help identify performance bottlenecks, while static analyzers can catch inefficiencies early in the development process.

#### Common Mistakes/Follow-Ups
Misunderstanding Complexity: Failing to recognize that contains checks on a list are (O(n)) and repeated within nested loops leads to (O(n^2)).
Ignoring Data Structure Choices: Not opting for more efficient data structures like sets can lead to unnecessary complexity.
Lack of Profiling: Without runtime monitoring, it's hard to catch issues that only become apparent under load or during specific operations.

**Comparison:**

```
ncalls	tottime	percall	cumtime	percall
1	0.53	0.53	0.53	0.53
......
```

**Code:**

```python
import cProfile

def check_performance():
    lst = [i for i in range(10000)]
    target = 5000
    # Simulate repeated contains checks
    for _ in range(100):
        for j in lst:
            if j == target:
                pass

cProfile.run('check_performance()')
```

**Example input / output:**

```text
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
......
```

**Data:**

```json
{
  "lst": [i for i in range(10000)],
  "target": 5000
}
```

**Flow:**

1. contains checks
2. Code Review
3. Check for nested loops
4. Flag inefficiencies
5. Static Analysis Tools
6. Integrate PyLint, ESLint
7. Runtime Monitoring
8. Instrument critical sections
9. Use profilers: cProfile, gprof,jvisualvm

