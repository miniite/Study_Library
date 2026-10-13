---
type: "practice"
difficulty: "Basic"
---

# What is cosine similarity and why is it commonly used to rank retrieved chunks?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Chunk Size and Retrieval Quality](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Chunk_Size_and_Retrieval_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cosine similarity is a measure of similarity between two non-zero vectors in an inner product space that can be used to rank retrieved chunks based on their semantic similarity. It is often employed because it effectively captures how aligned or similar the content of two pieces of text are, independent of their magnitude.

To understand why cosine similarity is commonly used for ranking retrieved chunks, let's break down its concept and application step-by-step from first principles.

**Code:**

```python
from sklearn.metrics.pairwise import cosine_similarity

# Example vectors representing chunks
chunk1 = [0.3, 0.5, 0.2]
chunk2 = [0.4, 0.4, 0.2]

# Calculating cosine similarity between the two vectors
similarity_score = cosine_similarity([chunk1], [chunk2])
print(f"Cosine Similarity: {similarity_score[0][0]}")
```

**Example input / output:**

```

Input:

PYTHON
Copy
chunk1 = [0.3, 0.5, 0.2]
chunk2 = [0.4, 0.4, 0.2]
similarity_score = cosine_similarity([chunk1], [chunk2])

Output:

CODE
Copy
Cosine Similarity: 0.9798
```

**Flow:**

1. Vectorize Chunks
2. Calculate Cosine Similarity
3. Rank Similarity Scores
4. Select Most Relevant Chunk

