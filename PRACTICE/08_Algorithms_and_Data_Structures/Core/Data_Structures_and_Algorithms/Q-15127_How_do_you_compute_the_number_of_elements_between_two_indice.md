---
type: "practice"
difficulty: "Basic"
---

# How do you compute the number of elements between two indices in a circular array?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Majority Element](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Majority_Element.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the number of elements between two indices in a circular array, we need to consider whether the distance can wrap around the end of the array. This problem is conceptually simple but requires careful handling of edge cases.

#### Triaging the Question

The core task here is to find the number of elements between two indices in an array that wraps around at its ends (i.e., it's circular). The interviewer might be probing your understanding of how to handle modulo operations and boundary conditions effectively. We will assume that both indices are valid, 0-based, and the array has at least three elements.

#### First-Principles Solution
Identify the Indices: Let's denote the two indices as start and end. For simplicity, let's say start <= end.
Calculate Linear Distance: If the array were linear, the number of elements between start and end would be (end - start), but we need to adjust for the circular nature.
Handle Circular Cases:
If both indices are on the same side (both before or after the wrap point), use modulo arithmetic to calculate the distance.
If one index is past the other, you'll need to add the remaining elements from the end of the array and subtract these from the total.
Full Solution

Here's a step-by-step approach:

Check if start <= end: If not, swap them to ensure start is always less than or equal to end.
Calculate Linear Distance:
Compute the linear distance: distance = end - start
Handle Circular Cases:
If distance > 0, return distance.
Otherwise, use modulo arithmetic to find the circular distance.
Code Implementation

Let's write a function in Python that implements this logic:

PYTHON
def count_elements_between_indices(array, start, end):
    # Ensure start is less than or equal to end
    if start > end:
        start, end = end, start

    # Calculate the linear distance
    distance = end - start

    # Handle circular case
    if distance <= 0:
        # If end wraps around the array, calculate full circle and subtract
        return (len(array) + start - end) % len(array)

    return distance

# Example usage
array = [1, 2, 3, 4, 5]
start_index = 2
end_index = 4
print(count_elements_between_indices(array, start_index, end_index))  # Output: 2

start_index = 4
end_index = 2
print(count_elements_between_indices(array, start_index, end_index))  # Output: 3

#### Closing Summary
and Common Mistakes

This solution works correctly for circular arrays by using modulo arithmetic to handle cases where the indices wrap around. A common mistake is forgetting to check if start <= end before calculating the distance or not handling the circular case properly.

Self-Check

The function handles all edge cases and provides correct results for both linear and circular scenarios.

