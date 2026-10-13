---
type: "practice"
difficulty: "Intermediate"
---

# Find all triplets in an array that sum to zero, and explain how sorting avoids duplicates.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array In-place Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array_In-place_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding all unique triplets in an array that sum to zero using a sorted array, we can leverage the properties of sorted arrays to efficiently avoid duplicate solutions. The basic approach involves sorting the array and then iterating through it while using two pointers to find valid triplets.

#### Triage

The question is asking for an algorithm to identify all unique triplets in an array whose sum equals zero. Sorting the array first helps in skipping over duplicates and reducing unnecessary checks, ensuring we only consider valid solutions once. The core of the solution involves iterating through the array with one pointer while using two other pointers (left and right) to find pairs that complement the current element.

Step-by-Step Solution
Sort the Array: Begin by sorting the array. This step is crucial as it allows us to use the two-pointer technique effectively.
Iterate Through the Array: Use a for loop to iterate through the array, treating each element as the first element of a potential triplet.
Two-Pointer Technique:
For each element at index i, initialize two pointers: one at the next position (left = i + 1) and another at the end of the array (right = n - 1).
Calculate the sum of the elements at indices i, left, and right.
If the sum is zero, add the triplet to the result.
If the sum is less than zero, increment the left pointer to increase the sum.
If the sum is greater than zero, decrement the right pointer to decrease the sum.
Skip Duplicates: To avoid adding duplicate triplets, skip over elements that are equal to the previous ones.

#### Example

#### Cons
ider the array [-1, 0, 1, 2, -1, -4]. After sorting, it becomes [-4, -1, -1, 0, 1, 2].

For the first element -4:
left = 1, right = 5 (sum is not zero).
Move to the next unique element -1 and repeat the process.
For the second -1:
left = 2, right = 5.
Check for sum, increment or decrement pointers accordingly.

#### Closing Summary

Sorting the array helps in efficiently managing duplicates by ensuring that we only consider unique solutions. The two-pointer technique, combined with sorting, allows us to find all zero-sum triplets without redundant checks, making the solution both efficient and effective.

#### Common Mistakes/Follow-Ups
Mistake: Not handling duplicate elements properly can lead to inclusion of invalid or repeated results.

#### Follow-Up
Discussing optimizations for very large datasets or considering different data types (e.g., floating-point numbers) might be relevant in certain contexts.

