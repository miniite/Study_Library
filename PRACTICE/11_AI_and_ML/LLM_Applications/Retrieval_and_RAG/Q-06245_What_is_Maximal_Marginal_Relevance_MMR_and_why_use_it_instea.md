---
type: "practice"
difficulty: "Deep"
---

# What is Maximal Marginal Relevance (MMR) and why use it instead of plain top-k?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Maximal Marginal Relevance (MMR) is a strategy used in information retrieval to select items that are most informative while avoiding redundancy. In contrast to simply selecting the top (k) items, MMR ensures that each added item contributes significantly by balancing the relevance and diversity of the results.

To understand why we use MMR over plain top-(k), let's first break down what these methods entail:

Top-k Selection: This is a straightforward method where you simply select the (k) most relevant items based on some scoring function, such as cosine similarity or relevance score. While this approach is simple and effective for retrieving highly relevant items, it often leads to redundancy because similar items with slightly different relevance scores might be included.

MMR: This strategy aims to maximize the diversity of selected items while maintaining their relevance. It works by selecting an item that maximizes its marginal gain in terms of both relevance and diversity relative to the previously chosen items. MMR ensures that each added item provides new, valuable information, reducing redundancy.

Here’s a step-by-step breakdown of how MMR works:

Initialization: Start with an empty set (R) (the result set).
Iteration:
Select an item (i) from the candidate pool such that it maximizes both its relevance to the query and the reduction in diversity among items already selected.
Add this item to (R).
Repeat until (k) items are selected or no further improvement is possible.

Now, let's see why MMR is preferred over plain top-(k):

Reduction of Redundancy: By focusing on both relevance and diversity, MMR ensures that the selected items provide a broader range of information. For example, if you're searching for images related to cats, using MMR might give you different breeds, cat behaviors, and even some general nature scenes, while top-(k) could just repeat similar photos of indoor domestic cats.

Improved Relevance: MMR can better capture the underlying structure of the data by selecting items that are not only relevant but also sufficiently different from each other. This is particularly useful in scenarios where the query or task requires a diverse set of results, such as content recommendation systems.

Scalability and Efficiency: While top-(k) might seem simple, it can quickly become inefficient when dealing with large datasets. MMR, though potentially more complex to implement, often provides better performance by reducing the number of redundant items in the output.

Let's consider a concrete example:

Suppose you have 100 images related to cats, and you want to select 5 images.
Top-(k) might return 5 similar indoor scenes with domestic cats.
MMR would likely choose one indoor scene, another outdoor scene, a close-up of a cat's face, a family petting their cat, and an image of a cat playing. This set is more diverse yet still relevant.

In summary, MMR excels when you need to balance relevance with diversity, ensuring that the selected items provide comprehensive coverage while avoiding redundancy. It’s particularly useful in applications like content recommendation systems where both the quality and variety of results are crucial.

**Comparison:**

| Item ID | Features    |
|---------|-------------|
| 1       | [0.15, 0.25, 0.3] |
| 2       | [0.18, 0.27, 0.33] |
| 3       | [0.2, 0.3, 0.35]  |
| 4       | [0.22, 0.29, 0.36]|

**Code:**

```python
# Example implementation of MMR using a simple scoring function

def mmr(query_vector, candidate_pool, k):
    # Initialize selected items
    R = []

    while len(R) < k:
        max_mmr_score = -1
        best_item = None

        for item in candidate_pool:
            relevance_score = similarity_score(item, query_vector)

            diversity_scores = [similarity_score(i, item) for i in R]
            if not diversity_scores:  # No previous items selected
                diversity_score = 0
            else:
                max_diversity = max(diversity_scores)
                if max_diversity == 1:  # Perfect match, avoid redundancy
                    continue
                diversity_score = (1 - max_diversity) / len(R) + 1 / (len(R) + 1)

            mmr_score = relevance_score * diversity_score

            if mmr_score > max_mmr_score:
                max_mmr_score = mmr_score
                best_item = item

        R.append(best_item)

    return R

# Helper functions for scoring
def similarity_score(item, query_vector):
    # Placeholder function; actual implementation depends on the data type and features
    pass
```

**Example input / output:**

```text
# Example input and output for MMR

candidate_pool = [
    {"id": 1, "features": [0.15, 0.25, 0.3]},
    {"id": 2, "features": [0.18, 0.27, 0.33]},
    {"id": 3, "features": [0.2, 0.3, 0.35]},
    {"id": 4, "features": [0.22, 0.29, 0.36]}
]

k = 5
query_vector = [0.1, 0.2, 0.3]
selected_items = mmr(query_vector, candidate_pool, k)
print("Selected items:", selected_items)
```

**Data:**

```json
{
  "query_vector": [0.1, 0.2, 0.3],
  "candidate_pool": [
    {"id": 1, "features": [0.15, 0.25, 0.3]},
    {"id": 2, "features": [0.18, 0.27, 0.33]},
    {"id": 3, "features": [0.2, 0.3, 0.35]},
    {"id": 4, "features": [0.22, 0.29, 0.36]}
  ]
}
```

**Flow:**

1. Start
2. Is R empty?
3. R is not empty
4. Select item with max MMRscore
5. R is empty
6. Choose the most relevant item
7. Add selected item to R
8. Update diversity scores foritems in R
9. Is |R| < k?
10. Yes, go back to B
11. No, return R as result

