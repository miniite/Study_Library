---
type: "practice"
difficulty: "Intermediate"
---

# An admin bulk-action endpoint applies an operation to selected rows but re-reads related config per row (O(rows x config-lookups)); how do you hoist the invariant lookup out and prove the complexity win?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity vs Real Performance](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_vs_Real_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: The goal is to optimize an admin bulk-action endpoint by reusing a configuration look-up that is currently performed per row, reducing redundant operations. This optimization involves hoisting the invariant lookup outside of the loop and proving the time complexity win.

#### Triage
The question probes your ability to identify inefficiencies in code, understand data structures, and refactor for performance improvements. We assume you are familiar with Big O notation and can analyze algorithms. The solution will involve analyzing the current complexity, identifying where redundant operations occur, refactoring to hoist the invariant lookup, and then proving the time complexity improvement.

#### First-Principles Solution
Let's start by understanding the current state of affairs:

Current Complexity Analysis:

The operation is performed on rows rows.
For each row, a configuration look-up (let’s denote this as config-lookup) is required.
This leads to a time complexity of O(rows x config-lookups).

Identifying Redundancies:

If the configuration data does not change per row and remains constant throughout the operation, we can hoist the look-up outside the loop.

Refactoring for Optimization:

Collect all required configurations at once before starting the loop.
Use a mapping or dictionary to quickly retrieve the configuration by key during the main operation.

Time Complexity Calculation:

Time complexity of collecting all configurations: O(config-lookups).
Time complexity of processing rows without look-ups: O(rows).

Thus, the overall time complexity becomes O(rows + config-lookups), which is a significant improvement over the original O(rows x config-lookups) if config-lookups is much smaller than rows.

Space Complexity Consideration:
Additional space required for storing all configurations in memory.

#### Closing Summary
By hoisting the invariant configuration look-up, we reduce redundant operations and improve overall performance. The key idea is to recognize when a lookup can be done once and reused, leading to O(rows + config-lookups) complexity versus O(rows x config-lookups).

#### Common Mistakes/Follow-Ups

Misidentifying Invariants: Failing to recognize that the configuration does not change per row.
Overlooking Space Complexity: Ignoring the extra space needed for storing configurations in memory, which might be a concern in resource-constrained environments.
Ignoring Edge Cases: Not considering scenarios where config-lookups are significantly smaller than rows, leading to minimal performance gains.

**Comparison:**

```
Original Complexity	Optimized Complexity
O(rows x config-lookups)	O(rows + config-lookups)
```

**Code:**

```python
# Original Code
def bulk_action(rows):
    for row in rows:
        config = get_config_for_row(row)  # O(config-lookup)
        process_row(row, config)          # O(1)

# Optimized Code
configs = {row['id']: get_config_for_row(row) for row in rows}  # O(rows * config-lookup)
for row in rows:
    process_row(row, configs[row['id']])  # O(config-lookup)
```

**Example input / output:**

```text
Input:
- `rows`: List of dictionaries representing rows with an 'id' key.

Output:
- Configurations are collected for all rows before processing.
- Each row's config is retrieved from the dictionary during processing.
```

**Data:**

```json
[
    {"id": 1, "name": "Row1"},
    {"id": 2, "name": "Row2"}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is configuration constant perrow?
5. Collect all configurations
6. Process rows using configs
7. Perform look-up for each row
8. Process row with config
9. End

