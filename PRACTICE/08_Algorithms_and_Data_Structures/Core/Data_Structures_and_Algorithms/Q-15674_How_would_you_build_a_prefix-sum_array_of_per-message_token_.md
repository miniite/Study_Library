---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a prefix-sum array of per-message token counts so you can answer 'how many tokens in messages i..j' in O(1)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to construct a prefix-sum array that allows us to quickly answer "how many tokens are there in messages from index (i) to (j)" with constant time complexity (O(1)). The key idea is to precompute the cumulative token counts for all possible message ranges. Let's break this down step-by-step.

#### Triage
Understanding the Problem

The question asks us to build a prefix-sum array where each element at index (i) stores the total number of tokens in messages up to and including message (i). This allows us to efficiently compute the sum of token counts for any range ([i, j]) by simply subtracting two precomputed values.

#### Assumptions
Messages are indexed starting from 0.
Each message is represented as a list of tokens (words or characters).
We have (n) messages to process.

#### First-Principles Solution

To achieve the desired complexity, we need to preprocess our data in such a way that any query can be answered with just two lookups and a subtraction. Here’s how we can do it:

Initialize an Array of Token Counts: Create an array token_counts where each element at index (i) holds the total number of tokens in messages up to and including message (i).
Compute Prefix Sums: Iterate through all messages, accumulating token counts into the token_counts array.
Querying the Array: For any range ([i, j]), the result can be computed as token_counts[j] - token_counts[i-1].
Step-by-Step Solution

Let's walk through a dry-run example with some sample data:

Sample Data
PYTHON
messages = [
    ["hello", "world"],
    ["how", "are", "you"],
    ["nice", "to", "meet", "you"]
]

Initialize the Token Count Array:

token_counts will have one more element than the number of messages, filled with zeros.

Compute Prefix Sums:

For message 0: tokens = 2 (["hello", "world"])
token_counts[0] = 2
For message 1: tokens = 3 (["how", "are", "you"])
token_counts[1] = 2 + 3 = 5
For message 2: tokens = 4 (["nice", "to", "meet", "you"])
token_counts[2] = 5 + 4 = 9

So, the token_counts array becomes:

PYTHON
token_counts = [0, 2, 5, 9]
Answer Queries:
For the range ([1, 2]):
token counts in messages from index 1 to 2: token_counts[2] - token_counts[0] = 9 - 2 = 7
For the range ([0, 2]):
token counts in messages from index 0 to 2: token_counts[2] - token_counts[-1] = 9 - 0 = 9

#### Closing Summary
and Common Mistakes/Follow-Ups
Correctness: Ensure that the array indices are handled correctly, especially when querying ranges.
Efficiency: The prefix-sum approach ensures constant time complexity for each query, making it highly efficient.
Edge Cases: What if a message is empty? How do we handle negative indices or out-of-bound queries?
Scalability: Consider how this solution would scale with a large number of messages.
Code Example
PYTHON
def build_prefix_sum(messages):
    n = len(messages)
    token_counts = [0] * (n + 1)

    for i in range(n):
        tokens_in_message_i = len(messages[i])
        token_counts[i + 1] = token_counts[i] + tokens_in_message_i

    return token_counts

def query_prefix_sum(token_counts, i, j):
    if i < 0 or j > len(token_counts) - 2:
        raise IndexError("Query out of bounds")

    return token_counts[j + 1] - token_counts[i]

# Example Usage
messages = [["hello", "world"], ["how", "are", "you"], ["nice", "to", "meet", "you"]]
token_counts = build_prefix_sum(messages)
print(query_prefix_sum(token_counts, 0, 2))  # Output: 9
Self-Check

The solution is correct and complete.

**Code:**

```python
def build_prefix_sum(messages):
    n = len(messages)
    token_counts = [0] * (n + 1)

    for i in range(n):
        tokens_in_message_i = len(messages[i])
        token_counts[i + 1] = token_counts[i] + tokens_in_message_i

    return token_counts

def query_prefix_sum(token_counts, i, j):
    if i < 0 or j > len(token_counts) - 2:
        raise IndexError("Query out of bounds")

    return token_counts[j + 1] - token_counts[i]

# Example Usage
messages = [["hello", "world"], ["how", "are", "you"], ["nice", "to", "meet", "you"]]
token_counts = build_prefix_sum(messages)
print(query_prefix_sum(token_counts, 0, 2))  # Output: 9
```

**Example input / output:**

```text
token_counts = [0, 2, 5, 9]
query_prefix_sum(token_counts, 1, 2) -> 7
query_prefix_sum(token_counts, 0, 2) -> 9
```

**Data:**

```python
messages = [["hello", "world"], ["how", "are", "you"], ["nice", "to", "meet", "you"]]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is i < 0 or j >len(token_counts)-1?
5. Raise IndexError
6. Compute query result astoken_counts[j+1] -token_counts[i]
7. End

