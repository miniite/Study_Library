---
type: "practice"
difficulty: "Intermediate"
---

# An algorithm's runtime graph looks linear at first but curves sharply upward as n grows; how do you name that complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this problem, we're looking to classify an algorithm’s time complexity when it initially appears linear but eventually becomes more complex as the input size (n) increases significantly. This is a common scenario where the initial behavior might suggest a simpler complexity class, but as (n) grows larger, the true complexity starts to dominate.

#### Triage

The question is essentially asking us to identify and name an algorithm's time complexity that transitions from linear ((O(n))) to something more complex. We need to understand how such complexities are typically classified and what they represent in terms of performance behavior.

#### First-Principles Solution

Let’s start by considering the basic definitions:

Linear Complexity ((O(n))): The runtime grows proportionally with the input size.
Quadratic Complexity ((O(n^2))): The runtime is proportional to the square of the input size, leading to a parabolic increase in time as (n) increases.

Given that the initial graph looks linear but sharply curves upward as (n) grows, we can infer that the algorithm transitions from (O(n)) to (O(n^2)). This transition point is where the quadratic nature of the algorithm starts to dominate. Therefore, the overall complexity can be classified as (O(n^2)).

Reasoning Step-by-Step
Initial Linear Behavior: For small values of (n), the runtime might look like (f(n) = cn + d), where (c) and (d) are constants.
Transition Point: As (n) increases, another factor starts to dominate the function, transforming it into a quadratic form.
Dominant Term: The term with the highest power of (n) will dominate as (n) grows large. In this case, that is (n^2).

#### Closing Summary

The complexity class we are dealing with here is (O(n^2)). This classification helps us understand that the algorithm’s performance degrades more significantly than a simple linear increase when the input size becomes large.

#### Common Mistakes
and Follow-Ups
Misidentifying Initial Linear Behavior: Sometimes, interviewees might misinterpret the initial linear behavior as the overall complexity.
Overlooking the Transition Point: It's important to recognize that the complexity can change based on the value of (n). Understanding this transition point is crucial.

#### Example Scenario

#### Cons
ider a sorting algorithm like Bubble Sort. Initially, for small arrays, it might appear to be relatively fast due to its linear behavior. However, as the array size increases significantly, the quadratic nature starts to dominate, making the performance degrade rapidly.

**Code:**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        # Flag to detect any swap happened
        swapped = False
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        # If no two elements were swapped by inner loop, then break
        if not swapped:
            break
    return arr
```

**Example input / output:**

```text
Input: [64, 34, 25, 12, 22, 11, 90]
Output: [11, 12, 22, 25, 34, 64, 90] (sorted array)
```

**Data:**

```text
No specific data is required for this problem, but a sample input could be an array of integers.
```

**Flow:**

1. Yes
2. No
3. Input Array
4. Initial Passes
5. Swapped?
6. (No swap, break)
7. Continue Inner Loop
8. Check Next Pass

