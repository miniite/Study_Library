---
type: "practice"
difficulty: "Basic"
---

# Find the missing number in an array containing 0 to n with one number absent.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Missing Number](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Missing_Number.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding the missing number in an array that contains all numbers from 0 to (n) except one, we can use a few different approaches. The most straightforward and efficient method involves leveraging mathematical properties and bitwise operations.

#### Triaging the Question

The core of this question is to identify a missing integer within a range (0 to n). The interviewer might be probing your understanding of basic algorithms, summation formulas, and bitwise manipulation techniques. We assume that the array contains integers from 0 to (n) with one number missing. Our task is to find the missing number.

#### First-Principles Solution

Summation Formula Approach:

The sum of all numbers from 0 to (n) can be calculated using the formula (\frac{n(n+1)}{2}).
Subtract the sum of the array elements from this total sum.
The result will be the missing number.

Bitwise XOR Approach:

Utilize the properties of the XOR operation to identify the missing number without needing extra space.
Step-by-Step Solution
Summation Formula Method
Calculate the expected sum (S) for numbers from 0 to (n). [ S = \frac{n(n+1)}{2} ]
Calculate the actual sum of elements in the array.
The missing number is: [ \text{missing} = S - \sum(\text{array}) ]
Bitwise XOR Method
Initialize a variable to store the result, starting with 0.
Iterate through both the range (0 to (n)) and the array simultaneously, performing XOR operations.
The final value will be the missing number.

#### Example
Walkthrough

Let's consider an example where the array is ([0, 1, 2, 4]), and we need to find the missing number in the range 0 to 4.

Using the summation formula:

Expected sum (S = \frac{4(4+1)}{2} = 10).
Actual sum of array elements: (0 + 1 + 2 + 4 = 7).
Missing number: (10 - 7 = 3).

Using the XOR method:

Initialize result to 0.
Perform XOR operations: (0 \oplus 1 \oplus 2 \oplus 4 \oplus (0 \oplus 1 \oplus 2 \oplus 3 \oplus 4) = 3).

#### Closing Summary
and Common Mistakes

Both methods are efficient, but the XOR method is particularly space-efficient. However, both have their edge cases:

Ensure that the array contains valid integers within the specified range.
Handle potential integer overflow in very large ranges.

#### Common mistakes
include failing to initialize variables correctly or incorrectly applying mathematical formulas.

#### Conclusion

In summary, finding the missing number can be effectively solved using either a summation formula or bitwise XOR. The choice between these methods depends on space and time constraints.

**Comparison:**

```
Method	Operation	Complexity
Summation	Calculate sums	O(n)
Bitwise XOR	Perform XOR ops.	O(n)
```

**Code:**

```python
def find_missing_number_summation(arr):
    n = len(arr) + 1
    expected_sum = n * (n - 1) // 2
    actual_sum = sum(arr)
    return expected_sum - actual_sum

def find_missing_number_xor(arr):
    result = 0
    for i in range(len(arr) + 1):
        result ^= i
    for num in arr:
        result ^= num
    return result
```

**Example input / output:**

| Input Array | n |
|-------------|---|
| [0, 1, 2, 4] | 4 |
| [3, 0, 1]   | 3 |
| Output       |
|--------------|
| 3            |
| 2            |

**Data:**

```json
[
    {"arr": [0, 1, 2, 4], "n": 4},
    {"arr": [3, 0, 1], "n": 3}
]
```

**Flow:**

1. Start
2. Calculate expected sum S
3. Calculate actual array sum
4. Subtract to get missingnumber
5. Initialize result to 0
6. Iterate through range andarray
7. XOR operations
8. Final XOR gives the missingnumber

