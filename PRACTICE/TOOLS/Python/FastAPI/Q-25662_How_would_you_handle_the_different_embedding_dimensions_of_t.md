---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle the different embedding dimensions of two models when configuring a vector store from FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle different embedding dimensions of two models when configuring a vector store in FastAPI, we need to ensure that both embeddings are compatible with each other or adapt them to be so. This involves understanding how vector stores work, integrating these vectors into a FastAPI application, and ensuring robustness against dimensionality mismatches.

Firstly, let's clarify what the question is probing: You're essentially looking at managing multiple embedding models in a single vector store using FastAPI. The core challenge here is to handle different dimensions of embeddings from various models, which could range from handling data preprocessing steps to configuring the vector store.

Let’s assume we have two embedding models: Model A with an output dimension of 768 and Model B with an output dimension of 512. We need to ensure that these vectors can coexist in a unified vector store.

Step-by-Step Solution

Understand the Embedding Models: Each model has its own embedding dimensions, which means their outputs are arrays of different lengths.

Model A: ( [768] )
Model B: ( [512] )

Identify Common Ground:

The first step is to check if there's a way to transform the embeddings from one model into the format expected by the other or vice versa.
If both models output dense vectors, we can look at techniques like dimensionality reduction (e.g., PCA) or augmentation (e.g., zero-padding).

Data Preprocessing:

Zero-padding: We can pad shorter embeddings to match the longest embedding length.
PYTHON
from typing import List

def pad_embeddings(embeddings: List[List[float]], max_len: int) -> List[List[float]]:
    return [e + ([0] * (max_len - len(e))) for e in embeddings]
Truncation: If the longer embedding can be truncated to match the shorter one, this might also work.
PYTHON
def truncate_embeddings(embeddings: List[List[float]], max_len: int) -> List[List[float]]:
    return [e[:max_len] for e in embeddings]

Configuration of Vector Store:

Ensure that both types of embeddings are stored using the same vector store configuration.
PYTHON
from fastapi import FastAPI, Depends, HTTPException
from langchain.vectorstores import FAISS

app = FastAPI()

@app.on_event("startup")
async def startup():
    global vector_store
    max_len = 768  # Assuming Model A is the reference model with the longest dimension
    embeddings_a = [e[:max_len] for e in embeddings_model_a]
    embeddings_b = pad_embeddings(embeddings_model_b, max_len)
    combined_embeddings = embeddings_a + embeddings_b
    vector_store = FAISS.from_embeddings(combined_embeddings, embedding_model=reference_embedding)

Query Handling:

Ensure that queries to the vector store can handle different dimensions gracefully.
PYTHON
@app.get("/search/{query}")
async def search(query: str):
    # Search logic here
    results = vector_store.similarity_search(query, k=3)
    return [r.page_content for r in results]

#### Closing Summary

To handle different embedding dimensions from multiple models when configuring a vector store with FastAPI, we need to perform data preprocessing steps like zero-padding or truncation. By ensuring that all embeddings are of uniform length and configured correctly within the vector store, we can maintain consistency and avoid issues during query handling.

#### Common Mistakes

Forgetting to handle dimensionality mismatches: Always ensure that input embeddings are compatible.
Not testing edge cases: Make sure you thoroughly test the system with different combinations of embeddings.

**Code:**

```python
from typing import List

def pad_embeddings(embeddings: List[List[float]], max_len: int) -> List[List[float]]:
    return [e + ([0] * (max_len - len(e))) for e in embeddings]

def truncate_embeddings(embeddings: List[List[float]], max_len: int) -> List[List[float]]:
    return [e[:max_len] for e in embeddings]
```

**Example input / output:**

```text
# Example input and output
embeddings_model_a = [[1, 2, 3], [4, 5, 6]]
embeddings_model_b = [[7, 8, 9, 10]]

max_len = 768

padded_embeddings = pad_embeddings(embeddings_model_b, max_len)
combined_embeddings = embeddings_model_a + padded_embeddings
```

