---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether to precompute embeddings offline and serve them from a store via FastAPI versus embedding on the fly, based on text cardinality and freshness needs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between precomputing embeddings offline and serving them from a store via FastAPI versus computing embeddings on the fly in an AI application using FastAPI, we need to consider several factors such as text cardinality (the number of unique texts), the freshness requirements of the data, performance considerations, and storage needs. The core idea is to balance between computation overhead, memory usage, and response time.

Firstly, let’s break down what we’re probing:

Text Cardinality: The size of your dataset.
Freshness Needs: How frequently new embeddings need to be computed.
Performance Requirements: Latency expectations for the application.
Storage Considerations: Memory and disk space usage.
Step-by-Step Analysis
1. Precompute vs On-the-Fly: What’s Probed

The interviewer is likely probing:

Your understanding of how embeddings work in FastAPI.
Your ability to evaluate trade-offs between precomputation and on-the-fly computation.
Your knowledge of the performance implications of each approach.
2. Assumptions

Let's make some assumptions for our analysis:

The application uses a moderate-sized dataset with varying text cardinality.
Freshness is a critical requirement, but not so frequent that it necessitates real-time embedding updates.
Performance and memory usage are significant concerns due to the resource constraints of the FastAPI service.
3. Precompute Embeddings Offline

#### Pros

Reduced Latency: Faster response times since embeddings are already computed.
Memory Efficiency: Can handle large datasets without the computational overhead during runtime.

#### Cons
istency: Ensures consistent embedding results across all queries.

#### Cons

Storage Requirements: More disk space is needed to store precomputed embeddings.
Frequent Updates: May require frequent updates if new texts are added frequently, which can be resource-intensive.
4. Compute Embeddings On the Fly

#### Pros

Lower Storage Costs: No need to store a large dataset of embeddings.
Flexibility: Can handle dynamic changes in the text corpus without precomputation.

#### Cons

Increased Latency: Higher computational overhead for each query, leading to slower response times.
Resource Intensive: More CPU and memory usage during runtime computations.
5. Practical Considerations

Given FastAPI’s performance characteristics:

If text cardinality is low to moderate (e.g., a few thousand texts), precomputation might be more efficient due to reduced overhead.
For very large datasets, on-the-fly computation could be necessary despite higher latency, as storage costs and complexity can become prohibitive.
6. Example Scenario

Let’s consider an example where we have around 10,000 unique texts:

Precomputation: Compute embeddings once offline using a tool like Gensim or SentenceTransformer.
FastAPI Service: Serve these precomputed embeddings via FastAPI endpoints with minimal computation overhead.

#### Closing Summary
and Mistakes to Avoid

In conclusion, the choice between precomputing embeddings offline versus computing them on-the-fly depends heavily on the specific use case. Precomputation is generally more efficient for smaller datasets or scenarios where response time is critical but storage is not a significant concern. On-the-fly computation is better suited for larger, dynamic text corpora with lower memory and storage requirements.

A common mistake is failing to consider all aspects of performance (latency, resource usage) and data management (storage). Always validate your decision through empirical testing in a realistic environment.

**Comparison:**

```
Text	Embedding (Example)
hello world	[0.1, -0.2, 0.3]
fastapi is awesome	[-0.4, 0.5, -0.6]
machine learning	[0.7, 0.8, -0.9]
```

**Code:**

```python
# Example FastAPI code for serving precomputed embeddings

from fastapi import FastAPI
import numpy as np

app = FastAPI()

@dataclass
class PrecomputedEmbeddings:
    embeddings: List[List[float]]

embeddings_store = PrecomputedEmbeddings(embeddings=[np.random.rand(128) for _ in range(10000)])

@app.get("/embedding/{text}")
async def get_embedding(text: str):
    # Assume some mapping from text to index is available
    idx = hash(text) % 10000
    return {"embedding": embeddings_store.embeddings[idx].tolist()}
```

**Example input / output:**

```text
Input:
GET /embedding/hello%20world

Output:
{
  "embedding": [0.1, -0.2, 0.3, ...]
}
```

**Data:**

```json
{
  "texts": [
    "hello world",
    "fastapi is awesome",
    "machine learning"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Text Input
4. Is Text in Cache?
5. Return PrecomputedEmbedding
6. Compute EmbeddingOn-the-Fly
7. Serve Embedding via API
8. Store New Embedding inCache

