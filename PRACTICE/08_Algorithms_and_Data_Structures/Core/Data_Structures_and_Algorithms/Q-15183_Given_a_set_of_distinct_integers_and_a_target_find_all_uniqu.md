---
type: "practice"
difficulty: "Intermediate"
---

# Given a set of distinct integers and a target, find all unique combinations summing to it.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Combinations](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Combinations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to generate all unique combinations of a given set of distinct integers that sum up to a specified target. The solution involves using backtracking, which is an efficient approach for exploring all possible combinations while avoiding duplicates.

#### Triage

The question asks us to find all unique combinations of numbers from a given list that add up to a target value. This implies we need to explore the power set of the input list and filter out those subsets whose sum equals the target. The key is ensuring uniqueness, which can be achieved by sorting the input array and pruning branches in the recursion.

#### First-Principles Solution
Sort the Input List: Sorting helps in handling duplicates efficiently.
Backtracking Approach: Use a recursive function to explore all possible combinations.
Pruning Duplicates: Skip over duplicate numbers during the search to avoid generating redundant results.

Here’s how we can implement this:

PYTHON
def combination_sum(candidates, target):
    result = []

    def backtrack(remaining, combo, start):
        if remaining == 0:
            result.append(list(combo))
            return
        elif remaining < 0:
            return

        for i in range(start, len(candidates)):
            candidate = candidates[i]
            # Skip duplicates
            if i > start and candidates[i] == candidates[i-1]:
                continue
            combo.append(candidate)
            backtrack(remaining - candidate, combo, i)
            combo.pop()

    candidates.sort()  # Sort to handle duplicates easily
    backtrack(target, [], 0)
    return result

# Example usage:
candidates = [2, 3, 5]
target = 8
print(combination_sum(candidates, target))  # Output: [[2, 2, 2, 2], [2, 3, 3], [3, 5]]

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution uses a backtracking approach with sorting to efficiently handle duplicates. By pruning branches where the remaining sum is less than zero and skipping over duplicate numbers, we ensure that only unique combinations are generated.

#### Common Mistakes

Not handling duplicates correctly can lead to redundant results.
Failing to sort the input list can complicate pruning logic.
Misunderstanding the problem statement might result in incorrect solution attempts (e.g., generating all possible subsets instead of specific summing combinations).

#### Follow-up
questions could involve optimizing for larger inputs, considering non-distinct integers, or handling edge cases like empty lists.

**Code:**

```python
def combination_sum(candidates, target):
    result = []

    def backtrack(remaining, combo, start):
        if remaining == 0:
            result.append(list(combo))
            return
        elif remaining < 0:
            return

        for i in range(start, len(candidates)):
            candidate = candidates[i]
            # Skip duplicates
            if i > start and candidates[i] == candidates[i-1]:
                continue
            combo.append(candidate)
            backtrack(remaining - candidate, combo, i)
            combo.pop()

    candidates.sort()  # Sort to handle duplicates easily
    backtrack(target, [], 0)
    return result

# Example usage:
candidates = [2, 3, 5]
target = 8
print(combination_sum(candidates, target))  # Output: [[2, 2, 2, 2], [2, 3, 3], [3, 5]]
```

**Example input / output:**

```text
candidates = [2, 3, 5]
target = 8
result = combination_sum(candidates, target)
print(result)  # Output: [[2, 2, 2, 2], [2, 3, 3], [3, 5]]
```

