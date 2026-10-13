---
type: "practice"
difficulty: "Intermediate"
---

# In JavaScript, what is the complexity of Array.prototype.indexOf and why does chaining it in a loop hurt?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the complexity of Array.prototype.indexOf in JavaScript and understand why chaining it in a loop can be problematic, we need to break down how these operations work under the hood. Let's start by defining the key concepts and then dive into a detailed explanation.

#### Triaging the Question

The question is asking about the time and space complexity of Array.prototype.indexOf when used within a loop. The main concern here is that chaining this method inside a loop can lead to inefficiencies, particularly in terms of time complexity.

Key Concepts
Time Complexity: This measures how long an algorithm takes to run as the size of the input data increases.
Space Complexity: This refers to the amount of memory used by an algorithm during its execution.

#### First-Principles Solution
Array.prototype.indexOf

The time complexity of Array.prototype.indexOf is ( O(n) ), where ( n ) is the length of the array. This means that in the worst case, it will traverse the entire array to find a match.

JAVASCRIPT

#### cons
t arr = [1, 2, 3, 4, 5];
arr.indexOf(3); // O(n)
Chaining indexOf in a Loop

When you chain Array.prototype.indexOf inside a loop that iterates over the same array, the operation is repeated for each element. If the loop runs ( m ) times and each call to indexOf takes ( O(n) ), then the total time complexity can become ( O(m \times n) ).

JAVASCRIPT

#### cons
t arr = [1, 2, 3, 4, 5];
for (let i = 0; i < arr.length; i++) {

#### cons
t index = arr.indexOf(3);
}
// This loop runs O(n) * O(n) = O(n^2)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, chaining Array.prototype.indexOf in a loop can significantly increase the time complexity. While each call to indexOf is ( O(n) ), if it's called within a loop that iterates over the same array multiple times, the overall complexity can become quadratic. To avoid this, consider using other approaches like setting up an index map or using a single pass to find all occurrences.

#### Common Mistakes
Not Understanding the Time Complexity: Many developers might not fully grasp how repeated calls in a loop can accumulate time complexity.
Overlooking Simplifications: There are often simpler and more efficient ways to achieve the same goal, such as using reduce, map, or filter.

#### Follow-Ups
Alternative Solutions: Can you suggest an alternative approach that avoids chaining indexOf in a loop?

#### Trade-Offs
What trade-offs should be considered when deciding between simplicity and performance?

**Comparison:**

```
Iteration	Array Length (n)	Time Complexity per Call	Total Time Complexity
1	5	O(5)	-
...	...	...	...
m	5	O(5)	O(m * n)
```

**Code:**

```javascript
// Example of a simple loop with indexOf
const arr = [1, 2, 3, 4, 5];
for (let i = 0; i < arr.length; i++) {
    const index = arr.indexOf(3);
    console.log(index); // This operation is O(n) for each call in the loop
}
```

**Example input / output:**

```text
Input:
- Array: [1, 2, 3, 4, 5]

Output (for each iteration in the loop):
- Index of 3: -1

Time Complexity Analysis:
- Each call to indexOf is O(n)
- Total time complexity for the loop: O(m * n)
```

**Data:**

```json
{
    "array": [1, 2, 3, 4, 5]
}
```

**Flow:**

1. Yes
2. No
3. Loop Iterations
4. Call to Array.prototype.indexOf
5. Is the element found?
6. Return index
7. Continue loop
8. Log or use index
9. Next iteration

