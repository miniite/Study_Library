---
type: "practice"
difficulty: "Deep"
---

# You must assemble a RAG prompt that maximizes retrieved-chunk relevance while staying under the token budget; why is this a 0/1 knapsack, and when do you drop the DP for a greedy value-density heuristic in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To assemble an RAG (Retrieval-Augmented Generation) prompt that maximizes retrieved-chunk relevance while staying under the token budget, we can model this as a 0/1 Knapsack problem. The goal is to select chunks of text such that their total tokens are within the budget and they collectively carry the highest possible information value.

#### Triaging the Question

The core challenge here is balancing the selection of relevant textual chunks while adhering to a token limit, which can be framed as a classic 0/1 Knapsack problem. In this context:

Items = Chunks of text.
Weights = Token counts of each chunk.
Values = Relevance scores (e.g., cosine similarity) or other measures indicating the information value.

#### Assumptions
and Context
Token Budget: A fixed number of tokens we can use for the prompt.
Relevancy Scores: Each chunk has a score representing its relevance to the query.
Greedy vs Dynamic Programming (DP): We need to decide whether to use dynamic programming (DP) or a greedy value-density heuristic in practice.

#### First-Principles Solution

The 0/1 Knapsack problem seeks to maximize the sum of values while keeping the weights under a certain limit. Here, we aim to optimize for relevance scores within the token budget. The dynamic programming approach is ideal for solving this as it ensures an optimal solution by considering all possible subsets.

Dynamic Programming (DP) Approach

We can use DP to solve this problem efficiently:

Initialization: Create a table dp where dp[i][j] represents the maximum relevance score achievable with i chunks and j tokens.
Filling the Table:
For each chunk, decide whether to include it or not.
If we include the chunk: dp[i][j] = dp[i-1][j-w[i]] + v[i], where w[i] is the token count and v[i] is the relevance score of chunk i.
If we don't include it: dp[i][j] = dp[i-1][j].
Result: The value at dp[n][budget] gives us the maximum relevance score.
Greedy Value-Density Heuristic

For real-time or high-throughput scenarios, a greedy approach might be preferred due to its simplicity and faster execution. The idea is to select chunks with higher value-to-weight ratio (value-density).

Compute Value-Density: For each chunk, compute v[i] / w[i].
Sort Chunks: Sort by value-density in descending order.
Select Chunks Greedily:
Add the next best chunk that fits into the remaining budget until no more chunks can be added.
Why Use DP?
Optimality Guarantee: DP ensures we find the optimal solution, whereas a greedy approach may miss some combinations of chunks that would yield higher relevance.
Complexity: While both approaches have similar time complexities (O(n * budget) and O(n * log n) respectively), DP is more robust for complex constraints.
When to Drop DP?

In production environments where real-time performance is critical, the overhead of dynamic programming might not be justifiable. Here are scenarios where a greedy approach could be suitable:

High Throughput Requirements: Real-time applications where quick decisions are needed.
Simplified Constraints: If the number of chunks or token budget constraints allow for simpler heuristics.

#### Closing Summary
and Common Mistakes

In summary, modeling the RAG prompt assembly as a 0/1 Knapsack problem provides an optimal solution through DP. However, in production, greedy algorithms can be used when real-time performance is critical and simplified constraints apply. Key mistakes include:

Overlooking the need for a token budget.
Ignoring relevance scores that could guide chunk selection.
Failing to consider both DP and greedy approaches and their trade-offs.

**Comparison:**

```
Chunk	Tokens	Relevance Score
1	50	0.8
2	70	0.9
3	60	0.7
```

**Code:**

```python
def knapsack_dp(chunks, token_budget):
    n = len(chunks)
    dp = [[0] * (token_budget + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        weight, value = chunks[i-1]
        for j in range(token_budget + 1):
            if j >= weight:
                dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight] + value)
            else:
                dp[i][j] = dp[i-1][j]

    return dp[n][token_budget]

def knapsack_greedy(chunks, token_budget):
    chunks.sort(key=lambda x: x[1]/x[0], reverse=True)  # Sort by value-density
    selected_chunks = []
    current_weight = 0

    for weight, value in chunks:
        if current_weight + weight <= token_budget:
            selected_chunks.append((weight, value))
            current_weight += weight

    return [(chunk[1], chunk[0]) for chunk in selected_chunks]
```

**Example input / output:**

```text
# Example input data
chunks = [(50, 0.8), (70, 0.9), (60, 0.7)]
token_budget = 120

# DP solution output
print(knapsack_dp(chunks, token_budget))  # Output: 1.48

# Greedy solution output
print(knapsack_greedy(chunks, token_budget))  # Output: [(70, 0.9), (50, 0.8)]
```

**Data:**

```json
[
  {"text": "Chunk 1", "tokens": 50, "relevance_score": 0.8},
  {"text": "Chunk 2", "tokens": 70, "relevance_score": 0.9},
  {"text": "Chunk 3", "tokens": 60, "relevance_score": 0.7}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is DP feasible?
5. Knapsack DP
6. Greedy Heuristic
7. Optimal Relevance Score
8. Selected Chunks
9. End

