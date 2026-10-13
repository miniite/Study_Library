---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a simple retrieval endpoint in FastAPI that returns the nearest embeddings to a query vector?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a simple retrieval endpoint in FastAPI, we'll focus on creating an API that takes a query vector and returns the nearest embeddings from a predefined set of vectors. This task involves setting up a FastAPI server, defining routes, handling input data, using cosine similarity to find the nearest neighbors, and ensuring efficient performance.

First, let's break down what the interviewer is probing: they want to see your understanding of FastAPI setup, familiarity with vector similarity metrics (like cosine similarity), and the ability to implement these concepts in a practical application. You'll need to make assumptions about the data format and size for simplicity, as well as considerations like performance and scalability.

Now, let's dive into building this endpoint from first principles:

Set Up FastAPI: We start by installing FastAPI and a dependency like numpy or scikit-learn for vector operations.

Define the Data Structure: Assume we have a precomputed set of embeddings stored in a NumPy array.

Create the Retrieval Function: This function will use cosine similarity to find the closest vectors to the query vector.

Build the FastAPI Endpoint: We'll create an endpoint that accepts a POST request with a JSON payload containing the query vector and returns the nearest embeddings.

Here's how you can implement this:

**Code:**

```python
from fastapi import FastAPI, HTTPException
import numpy as np

app = FastAPI()

# Example data: precomputed embeddings and their corresponding IDs
embeddings = np.random.rand(1000, 5)  # 1000 vectors in a 5-dimensional space
ids = [f"vec_{i}" for i in range(len(embeddings))]

@app.post("/nearest_embeddings/")
async def nearest_embeddings(query_vector: list):
    """
    Finds the nearest embeddings to the provided query vector.

    Args:
        query_vector (list): The input query vector as a list of floats.

    Returns:
        dict: A dictionary containing the IDs and corresponding vectors of the closest neighbors.
    """
    # Convert the query vector from list to numpy array for easy operations
    query_vector_np = np.array(query_vector)

    # Calculate cosine similarities between the query vector and all embeddings
    cos_sims = np.dot(embeddings, query_vector_np) / (np.linalg.norm(embeddings, axis=1) * np.linalg.norm(query_vector_np))

    # Get indices of the top 5 most similar vectors
    closest_indices = np.argsort(cos_sims)[-5:]

    # Prepare the response with nearest embeddings and their IDs
    nearest_embeddings_response = [
        {"id": ids[i], "embedding": embeddings[i].tolist()} for i in reversed(closest_indices)
    ]

    return {"nearest_embeddings": nearest_embeddings_response}
```

**Example input / output:**

```

For the input vector [0.2, 0.4, 0.6, 0.8, 1.0], the output might look like:

JSON
Copy
{
    "nearest_embeddings": [
        {"id": "vec_397", "embedding": [0.504, 0.362, 0.976, 0.361, 0.886]},
        {"id": "vec_549", "embedding": [0.717, 0.337, 0.537, 0.332, 0.455]},
        {"id": "vec_822", "embedding": [0.688, 0.449, 0.909, 0.819, 0.756]},
        {"id": "vec_681", "embedding": [0.385, 0.325, 0.828, 0.417, 0.987]},
        {"id": "vec_157", "embedding": [0.697, 0.296, 0.496, 0.623, 0.416]}
    ]
}
```

**Data:**

```python
# Example data schema
embeddings = np.random.rand(1000, 5)  # 1000 vectors in a 5-dimensional space
ids = [f"vec_{i}" for i in range(len(embeddings))]
```

