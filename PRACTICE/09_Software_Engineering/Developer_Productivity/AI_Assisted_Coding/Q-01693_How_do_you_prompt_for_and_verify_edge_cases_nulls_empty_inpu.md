---
type: "practice"
difficulty: "Intermediate"
---

# How do you prompt for and verify edge cases (nulls, empty input, overflow) in AI-generated code?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI-assisted coding tools like Copilot or similar systems, it's crucial to ensure that the generated code handles edge cases effectively. This involves prompting users for specific scenarios and then verifying those implementations through a series of checks. Here’s how you can approach this:

Prompting for Edge Cases: Start by identifying common edge cases such as null inputs, empty collections, or potential overflow conditions. For example, if working with a function that processes an array, prompt the user to consider what happens when the array is empty.

Verification through Checks and Tests: Once the code has been generated, it’s important to verify its correctness by implementing explicit checks within the code itself. This can be done programmatically or by using test frameworks to ensure edge cases are handled correctly.

#### Trade-offs
and Considerations: There's a balance between over-optimizing for edge cases and keeping the code maintainable. Overly complex checks might add unnecessary overhead, but neglecting them could lead to runtime errors.

**Code:**

```python
# Example of prompting and verifying an edge case in Python

def process_array(arr):
    if not arr:
        # Edge Case 1: Empty Array
        raise ValueError("Input array cannot be empty")

    result = []
    for item in arr:
        try:
            # Perform some operation that might fail with overflow
            processed_item = complex_operation(item)
            result.append(processed_item)
        except OverflowError:
            # Edge Case 2: Potential Overflow
            print(f"Overflow detected at item: {item}")

    return result

# Example of a function that could potentially cause an overflow
def complex_operation(x):
    return x * (x + 1) ** 1000

# Test the function with edge cases
try:
    process_array([1, 2, 3])
    print("No issues found")
except Exception as e:
    print(f"Exception: {e}")
```

**Example input / output:**

```text
Input: process_array([])
Expected Output: ValueError: Input array cannot be empty

Input: process_array([1, 2])
Output: No issues found

Input: complex_operation(999)
Output: Overflow detected at item: 999
```

**Flow:**

1. User Prompts
2. Prompt AI for functiondefinition
3. Check input types andconstraints
4. Generate Code
5. Verify Code with Edge CaseTests
6. Review and Refine Code

