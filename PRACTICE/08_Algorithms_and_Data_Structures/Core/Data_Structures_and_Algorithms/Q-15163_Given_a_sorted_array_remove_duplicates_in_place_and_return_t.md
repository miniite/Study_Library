---
type: "practice"
difficulty: "Basic"
---

# Given a sorted array, remove duplicates in place and return the new length.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Combinations](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Combinations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this problem, we are given a sorted array and asked to remove duplicates in place, meaning that no additional space should be used. The task is to return the new length of the array after removing the duplicates. Let's break down the solution step by step.

Firstly, the question asks us to do this in-place, which means modifying the original array directly without using extra space. We need to identify a way to track and store unique elements while maintaining their order relative to the initial array.

We will reason from first principles: since the input array is sorted, we can leverage that property to efficiently remove duplicates with a single pass through the array.

