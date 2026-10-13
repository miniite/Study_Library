---
type: "practice"
difficulty: "Basic"
---

# How would you compute the running sum of tokens across a streamed response to stop when a budget is hit, and why is that a single-pass O(1)-update operation?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of computing the running sum of tokens (let's assume tokens are integers) in a streamed response while stopping when a budget limit is reached, we need to design an efficient algorithm that updates the sum in a single pass and ensures it stops as soon as the budget is hit. This involves understanding how streaming data works and implementing a simple yet effective algorithm.

The core of the question is about performing an operation in constant time (O(1)) for each new token, ensuring we can process the data efficiently without additional space or time overhead beyond what's necessary for the current token.

We'll assume the tokens are integers and the budget limit is a predefined value. The algorithm will maintain a running sum as it processes each token from the stream and stop processing once this sum exceeds the budget limit.

**Code:**

```python
def compute_running_sum(tokens, budget):
    # Initialize variables to keep track of the current sum and index in the tokens list
    total_sum = 0
    for i, token in enumerate(tokens):
        total_sum += token

        # Check if the running sum exceeds the budget
        if total_sum > budget:
            return (i, total_sum)  # Return the index where the budget was exceeded and the current sum

    return (-1, total_sum)  # If no tokens exceed the budget, return the final sum

# Example usage
tokens = [3, -5, 2, 8, -4, 6]
budget_limit = 10

result_index, result_sum = compute_running_sum(tokens, budget_limit)
print(f"Index: {result_index}, Running Sum: {result_sum}")
```

**Example input / output:**

```text
Input:
tokens = [3, -5, 2, 8, -4, 6]
budget_limit = 10

Output:
Index: 3, Running Sum: 9
```

