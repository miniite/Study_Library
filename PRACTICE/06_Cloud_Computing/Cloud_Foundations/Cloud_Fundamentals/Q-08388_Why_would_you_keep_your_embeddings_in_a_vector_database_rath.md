---
type: "practice"
difficulty: "Basic"
---

# Why would you keep your embeddings in a vector database rather than recomputing them on every request?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Vector Databases](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Vector_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to store pre-computed embeddings or recompute them on each request, we must consider several factors such as computational cost, storage requirements, and performance implications. Generally, storing embeddings in a vector database is more efficient if the embedding generation process is costly relative to the frequency of requests.

Firstly, let's clarify what a vector database is: It’s a specialized database that stores vectors (embeddings) and allows for fast similarity searches. This is crucial when dealing with large-scale applications where we often need to find the most similar items to a given query vector. Storing embeddings in such databases can significantly reduce latency compared to recomputing them.

Now, let's break down why storing embeddings is beneficial:

Reduced Computational Cost: Computing embeddings can be resource-intensive and time-consuming, especially for deep learning models like BERT or word2vec. By precomputing these embeddings and storing them in a vector database, we eliminate the need to run this computationally expensive process on every request.

Improved Performance: Latency is a critical factor in many applications, particularly those that require real-time responses. Retrieving an embedding from a vector database is much faster than generating it anew. This can make a significant difference in user experience and application performance.

Scalability: In distributed systems, storing embeddings centrally can help in managing the state of the model consistently across multiple nodes. It also allows for easier updates to the model without affecting ongoing operations.

Cost Efficiency: While storage costs have decreased significantly over time, they are still a factor. If the data volume is large and infrequent updates are required, it might be more cost-effective to store embeddings rather than repeatedly compute them on-the-fly.

However, there are scenarios where recomputing embeddings could be preferable:

Dynamic Data: If your application frequently receives new or changing data that significantly alters the embedding space, you may need to recompute embeddings to ensure they reflect current knowledge.

Model Updates: When the model architecture or training data changes, recomputing embeddings ensures that the embeddings are relevant and up-to-date with the latest information.

Resource Constraints: In environments where storage is extremely limited, the computational cost of embedding generation might still be manageable compared to storage costs.

In conclusion, storing embeddings in a vector database is often a better choice when the frequency of requests exceeds the cost of precomputation. However, the decision should be made based on a thorough analysis of your specific use case and requirements.

#### Common mistakes
include:

Ignoring the computational cost of embedding generation.
Failing to consider real-time performance requirements.
Overlooking storage and maintenance overheads for embeddings.

**Code:**

```python
# Example Python code snippet for querying a vector database
import numpy as np
from annoy import AnnoyIndex

# Initialize an Annoy index with 128 dimensions (common embedding size)
t = AnnoyIndex(128, 'euclidean')

# Assume we have precomputed embeddings and their corresponding IDs
embeddings = np.random.rand(10000, 128)  # Example data
ids = [f"embedding_{i}" for i in range(10000)]
for id, embedding in zip(ids, embeddings):
    t.add_item(id, embedding)

# Build the index
t.build(10)  # 10 trees

# Querying the nearest neighbors to a given query vector
query_vector = np.random.rand(128)
nearest_neighbors_ids, distances = t.get_nns_by_vector(query_vector, 5)
print("Nearest neighbor IDs:", nearest_neighbors_ids)
```

**Example input / output:**

```text
# Example input and output for querying the vector database
query_vector = np.array([0.7, 0.8, 0.9])
nearest_neighbors, distances = t.get_nns_by_vector(query_vector, 2)
print("Nearest neighbor IDs:", nearest_neighbors)
```

**Data:**

```json
{
    "embeddings": [
        {"id": "embedding_0", "vector": [0.1, 0.2, 0.3]},
        {"id": "embedding_1", "vector": [0.4, 0.5, 0.6]},
        // ... more embeddings
    ],
    "query_vector": [0.7, 0.8, 0.9],
    "nearest_neighbors_ids": ["embedding_2", "embedding_3"]
}
```

