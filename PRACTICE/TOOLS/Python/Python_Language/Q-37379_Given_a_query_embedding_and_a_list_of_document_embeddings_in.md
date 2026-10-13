---
type: "practice"
difficulty: "Intermediate"
---

# Given a query embedding and a list of document embeddings in Python, how would you find the most similar document using NumPy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the most similar document given a query embedding and a list of document embeddings in Python using NumPy, we need to compute the cosine similarity between the query embedding and each document embedding, then select the one with the highest similarity score. This is a straightforward but important task for information retrieval systems.

#### Triage

The question asks us to find the most similar document by comparing embeddings. The key idea here is to use the cosine similarity metric since it measures the angle between vectors in high-dimensional space, making it suitable for embedding comparisons. We need to ensure that our implementation efficiently handles vector operations using NumPy and correctly normalizes the embeddings.

#### Solution
Normalize the Embeddings: Ensure both the query embedding and document embeddings are normalized.
Compute Cosine Similarities: Use np.dot or np.vdot for dot products, which aligns well with normalization.
Find Maximum Similarity: Identify the index of the maximum similarity score.

Let's go through this step-by-step:

Normalization:

We will normalize both the query and document embeddings to ensure they lie on the unit sphere, making cosine similarities comparable.

Cosine Similarity Calculation:

Use np.vdot for efficient computation of dot products between normalized vectors.
The cosine similarity is given by the dot product divided by the norms (which are 1 after normalization).

Finding Maximum Similarity:

Find the index of the maximum similarity score using NumPy's argmax function.

Here’s a detailed implementation:

PYTHON
import numpy as np

def find_most_similar(query_embedding, document_embeddings):
    # Normalize embeddings
    query_embedding = query_embedding / np.linalg.norm(query_embedding)
    normalized_doc_embeddings = [embedding / np.linalg.norm(embedding) for embedding in document_embeddings]

    # Compute cosine similarities
    similarities = [np.vdot(query_embedding, doc_embedding) for doc_embedding in normalized_doc_embeddings]

    # Find the index of maximum similarity
    max_index = np.argmax(similarities)

    return max_index

# Example usage:
query_embedding = np.array([0.1, 0.2, -0.3])
document_embeddings = [np.array([0.4, 0.5, -0.6]), np.array([-0.7, 0.8, 0.9])]
most_similar_index = find_most_similar(query_embedding, document_embeddings)
print(f"The most similar document is at index: {most_similar_index}")

#### Closing Summary

The solution involves normalizing the query and document embeddings to ensure they are unit vectors, which simplifies cosine similarity calculations. The np.vdot function efficiently computes dot products for these normalized vectors, and we use np.argmax to find the index of the most similar document.

#### Common Mistakes/Follow-Ups

Mistake: Not normalizing embeddings before computing similarities.

#### Follow-Up
Always normalize embeddings, especially when dealing with high-dimensional spaces.

Mistake: Using np.dot instead of np.vdot.

#### Follow-Up
np.vdot handles complex conjugation for non-real vectors correctly.

**Code:**

```python
import numpy as np

def find_most_similar(query_embedding, document_embeddings):
    # Normalize embeddings
    query_embedding = query_embedding / np.linalg.norm(query_embedding)
    normalized_doc_embeddings = [embedding / np.linalg.norm(embedding) for embedding in document_embeddings]

    # Compute cosine similarities
    similarities = [np.vdot(query_embedding, doc_embedding) for doc_embedding in normalized_doc_embeddings]

    # Find the index of maximum similarity
    max_index = np.argmax(similarities)

    return max_index

# Example usage:
query_embedding = np.array([0.1, 0.2, -0.3])
document_embeddings = [np.array([0.4, 0.5, -0.6]), np.array([-0.7, 0.8, 0.9])]
most_similar_index = find_most_similar(query_embedding, document_embeddings)
print(f"The most similar document is at index: {most_similar_index}")
```

