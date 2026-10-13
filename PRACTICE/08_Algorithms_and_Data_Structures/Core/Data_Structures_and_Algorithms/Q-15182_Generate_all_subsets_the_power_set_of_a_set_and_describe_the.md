---
type: "practice"
difficulty: "Intermediate"
---

# Generate all subsets (the power set) of a set and describe the recursion.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Subsets and Power Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Subsets_and_Power_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate all subsets of a set, also known as the power set, we can use a recursive approach. The key idea is that each element in the set has two choices: it either belongs to a subset or does not belong to any subset. This process continues until we have considered every possible combination.

#### Triage

The question asks us to implement a function generatePowerSet that takes a list of integers and returns all possible subsets, including the empty set and the set itself. The interviewer is likely probing your understanding of recursion and the combinatorial nature of the problem. We will assume the input is a non-empty list of unique integers.

#### First-Principles Solution

Let's break down the solution step-by-step:

Base Case: If the input list is empty, return a set containing an empty subset.
Recursive Step: Pick each element in the list and generate subsets by including it or excluding it. This can be achieved by splitting the problem into two smaller subproblems:
Generate all subsets that do not include the current element.
Generate all subsets that include the current element, which is achieved by adding the current element to each subset from the first step.

#### Implementation

Here's a Python implementation:

PYTHON
def generatePowerSet(nums):
    if not nums:
        return [[]]

    # Recursive call on the list without the first element
    subproblem = generatePowerSet(nums[1:])

    # Generate new subsets by including the first element in each subset from subproblem
    with_first_element = [[nums[0]] + subset for subset in subproblem]

    # Combine both sets of subsets: those that include and those that exclude the current element
    return subproblem + with_first_element

# Example usage
print(generatePowerSet([1, 2, 3]))
Explanation
The function generatePowerSet takes a list of integers.
If the input is empty, it returns [[]], which represents the base case (the power set of an empty set).
For each element in the input list, we recursively generate the power set for the rest of the elements and combine them with and without the current element.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The full solution involves a recursive approach where we build up all possible subsets by including or excluding each element. A common mistake is failing to handle the base case correctly, leading to infinite recursion. Another pitfall is not properly combining the results from the subproblems.

#### Follow-up
questions might include:

How would you modify this function if the input set could contain duplicate elements?
Can you optimize the solution for large sets of numbers?

