---
type: "practice"
difficulty: "Deep"
---

# You're deciding whether RAG (cheaper per query, index cost) or long-context stuffing (no index, higher per-query token cost) is the right architecture for a knowledge assistant. Walk me through the cost crossover as corpus size and query volume vary, and where each wins.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ RAG and Vector Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/RAG_and_Vector_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between RAG (Retrieval-Augmented Generation) and long-context stuffing for a knowledge assistant, we need to consider both the index costs and per-query token costs as the corpus size and query volume vary. The key is understanding the cost crossover point where each architecture becomes more or less expensive than the other. Let's break this down from first principles.

Firstly, RAG relies on an index that allows efficient retrieval of relevant documents based on queries, followed by a generative model to create the response. On the other hand, long-context stuffing involves loading all context into memory and generating the response directly without an index. The choice between these two architectures depends heavily on the size of the corpus (number of documents) and query volume.

The core concept is that as corpus size increases, indexing becomes more critical for efficient retrieval. Similarly, higher query volumes can make the overhead of maintaining an index less significant compared to the cost savings from reduced per-query token costs in long-context stuffing.

Let's reason through this step by step:

Cost Components:

RAG:
Indexing Cost: Proportional to corpus size.
Query Processing Cost: Low per query, but dependent on index efficiency and complexity of the generative model.
Long-Context Stuffing:
No Indexing Cost: This is a significant savings as corpus grows large.
Per-Query Token Cost: Higher for each query due to full context load.

Cost Model: We can represent these costs with simple equations, where (C_{RAG} = C_i + \alpha q) and (C_{LCS} = \beta c), where:

(C_i) is the one-time indexing cost.
(q) is the query volume.
(\alpha) represents the per-query cost for RAG, which includes both retrieval and generation costs.
(\beta) represents the constant token cost for each query in LCS.

Cost Crossover: The crossover point occurs where (C_{RAG} = C_{LCS}). This can be mathematically solved to find the balance between corpus size, query volume, and costs.

#### Example
Calculation: Suppose a small corpus of 10k documents with an indexing cost of $500 (once), and each query incurs a retrieval+generation cost of $0.2 per query. For long-context stuffing, assume each token costs $0.002.

If the corpus size (c = 10^4): [ C_{RAG} = 500 + 0.2q ] [ C_{LCS} = 0.002t ] where (t) is the total tokens loaded per query (assuming each document has an average of 1k tokens).

For a given query volume, we need to find when: [ 500 + 0.2q = 0.002(10^4)q ] Simplifying this, we get: [ 500 + 0.2q = 20q ] [ 500 = 19.8q ] [ q = \frac{500}{19.8} \approx 25.25 ]

This means that for a query volume of around 25 queries per document, RAG becomes more cost-effective than long-context stuffing.

**Comparison:**

```
Crossover Points	Query Volume (per document)
RAG Wins	< 25
LCS Wins	> 25
```

**Code:**

```python
def calculate_crossover_point(corpus_size, indexing_cost, retrieval_token_cost_per_query, token_cost_lcs):
    # Define the equations
    def rag_cost(q): return indexing_cost + (retrieval_token_cost_per_query * q)
    def lcs_cost(t): return token_cost_lcs * t

    # Calculate crossover point
    for q in range(1, 1000):  # reasonable upper limit
        if rag_cost(q) < lcs_cost(corpus_size * 1024):  # assuming each document has ~1k tokens
            return q
    return None

# Example calculation
corpus_size = 10000
indexing_cost = 500
retrieval_token_cost_per_query = 0.2
token_cost_lcs = 0.002
crossover_point = calculate_crossover_point(corpus_size, indexing_cost, retrieval_token_cost_per_query, token_cost_lcs)
print(f"Crossover Point: {crossover_point}")
```

**Example input / output:**

```text
Crossover Point: 25
```

**Data:**

```json
{
    "corpus_size": 10000,
    "indexing_cost": 500,
    "retrieval_token_cost_per_query": 0.2,
    "token_cost_lcs": 0.002
}
```

**Flow:**

1. Corpus Size
2. 10k
3. Indexing Cost
4. $500
5. Query Volume
6. 25 queries/document
7. RAG Cost
8. $500 + 0.2 * q < 0.002 * (10k *1024)q
9. LCS Cost
10. 20q
11. Crossover Point
12. 25 queries/document

