---
type: "practice"
difficulty: "Intermediate"
---

# A prod feature job does repeated 'contains' checks against a growing list that was never converted to a set, giving accidental O(n^2); how would you catch this class of bug in review or via profiling before it ships?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of repeated 'contains' checks against a growing list that can lead to an O(n^2) time complexity, we need to catch this class of bug both in code review and via profiling before it ships. Here’s how you would approach this:

Understanding the Problem: The main issue is that each contains check on a dynamically growing list (that has not been converted to a set) will traverse the entire list, leading to quadratic time complexity as the size of the list grows.

Code Review: During code review, look for any situation where a developer uses contains or similar operations on an unoptimized data structure. Ensure that lists are being managed efficiently and consider converting them to sets when needed, especially in scenarios with repeated membership checks.

Profiling and Analysis:

Static Analysis Tools: Use static analysis tools like SonarQube or Pylint to flag any potential issues where a contains check is performed on a list.
Dynamic Profiling: Run the application under load using profiling tools such as Python’s cProfile, Java’s VisualVM, or similar tools for other languages. Focus on identifying hotspots in the code that are repeatedly performing these checks.

#### First-Principles Solution

Convert to Set for Membership Checks: The simplest and most effective solution is to convert the list to a set when membership checks become frequent. Sets provide average O(1) time complexity for contains operations, significantly reducing the overall execution time.
Batch Optimization: If you can't afford the memory overhead of converting the entire list to a set at once, consider batch conversion or incremental optimization where sets are created only as needed.

#### Example
Code and Profiling:

PYTHON
# Original code using a growing list for contains checks
items = []
for item in some_input_list:
    if item not in items:  # This could lead to O(n^2) complexity
        do_something(item)
        items.append(item)

# Optimized code with set conversion
unique_items_set = set()
for item in some_input_list:
    if item not in unique_items_set:
        do_something(item)
        unique_items_set.add(item)

# Profiling the code to measure performance improvement
import cProfile

def profile_code(code):
    profile = cProfile.runctx(code, globals(), locals())
    with open('profile_results.txt', 'w') as f:
        profile.dump_stats(f)

original_code = "items = []; for item in some_input_list: if item not in items: do_something(item); items.append(item)"
optimized_code = "unique_items_set = set(); for item in some_input_list: if item not in unique_items_set: do_something(item); unique_items_set.add(item)"

profile_code(original_code)
profile_code(optimized_code)

#### Closing Summary
and Common Mistakes:

#### Common mistakes
include ignoring the performance implications of operations on dynamic lists, not recognizing when to convert data structures for efficiency.
Ensure that profiling tools are used effectively to identify bottlenecks before they become significant issues.

