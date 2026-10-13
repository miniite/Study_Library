---
type: "practice"
difficulty: "Intermediate"
---

# Find the longest substring with at most k distinct characters using a sliding window.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window Substrings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Substrings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we'll use a sliding window approach to find the longest substring in a given string that contains at most k distinct characters. The core idea is to maintain two pointers representing the current window's start and end, and use a hash map to keep track of the frequency of each character within the window. This solution has a time complexity of O(n), where n is the length of the input string.

#### Triage

The question asks us to find the longest substring with at most k distinct characters using a sliding window approach. The interviewer is probing your understanding of dynamic programming, hash maps, and efficient substring manipulation techniques. We'll assume that the input string is non-empty and can contain any printable ASCII characters.

#### Solution
From First Principles

Initialize Variables:

Use two pointers start and end to represent the current window.
A hash map charFrequency to keep track of character frequencies within the current window.
Variables maxLength, currentLength, and maxChars to store the maximum length found so far, the current length of the window, and the number of distinct characters in the current window.

Expand the Window:

Move the end pointer to expand the window by adding one character at a time.
Update the frequency of the added character in charFrequency.

Contract the Window:

If the number of distinct characters exceeds k, move the start pointer to shrink the window until there are at most k distinct characters.
Adjust the frequencies and contract the window by removing characters from the start.

Update Maximum Length:

After each expansion, check if the current length of the window is greater than maxLength. If so, update maxLength.

Return Result:

Once the end pointer has traversed the entire string, return maxLength as the result.

#### Closing Summary

This sliding window approach ensures that we efficiently find the longest substring with at most k distinct characters by balancing expansion and contraction of the window. The use of a hash map for frequency tracking allows us to handle character counts dynamically, making the solution both elegant and efficient.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking edge cases where the string length is less than k. In such cases, the entire string should be returned.

#### Follow-Up
Discussing how this approach can be extended to find the kth longest substring or using multiple sliding windows for more complex problems.

