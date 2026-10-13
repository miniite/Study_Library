---
type: "practice"
difficulty: "Deep"
---

# How would you implement request coalescing in a FastAPI embedding endpoint so N concurrent requests for the same text produce a single embedding computation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement request coalescing in a FastAPI embedding endpoint, we aim to ensure that multiple concurrent requests for the same text result in a single embedding computation. This is crucial to optimize computational resources and improve performance. Here’s how you would approach this:

#### Overview
We'll use caching mechanisms within FastAPI to store embeddings once they are computed. A distributed cache like Redis can help manage these computations efficiently.

#### Triage
The question is about optimizing the FastAPI endpoint by ensuring that identical requests do not trigger redundant embedding calculations, thereby improving resource utilization and latency performance.

#### First-Principles Solution
Use a caching layer to store embeddings once they are computed.
Implement a mechanism to check if an embedding for a given text already exists before performing the computation.
Use distributed locks or atomic operations to manage concurrent requests effectively.
Step-by-Step Implementation

Set Up Redis Cache:

PYTHON
import fastapi
from redis import asyncio as aioredis
from typing import Optional, Dict

app = fastapi.FastAPI()
redis_client: Optional[aioredis.Redis] = None

@app.on_event("startup")
async def startup():
    global redis_client
    redis_client = await aioredis.from_url("redis://localhost")

@app.on_event("shutdown")
async def shutdown():
    if redis_client is not None:
        await redis_client.close()

Check and Compute Embedding:

PYTHON
import numpy as np

EMBEDDING_CACHE: Dict[str, np.ndarray] = {}

@app.get("/embed/{text}")
async def get_embedding(text: str) -> Dict[str, any]:
    # Check if embedding is already cached in Redis
    key = f"embedding:{text}"
    result = await redis_client.get(key)

    if result:
        return {"embedding": np.loads(result.decode())}

    # If not cached, compute the embedding and cache it
    embedding = compute_embedding(text)  # Placeholder for your actual embedding computation logic

    EMBEDDING_CACHE[text] = embedding
    await redis_client.set(key, embedding.dumps())

    return {"embedding": embedding.tolist()}

Compute Embedding Function:

PYTHON
def compute_embedding(text: str) -> np.ndarray:
    # Placeholder for your actual embedding computation logic
    return np.random.rand(512)

#### Closing Summary
and Common Mistakes
Summary: By leveraging Redis caching, we ensure that each unique text request results in only one embedding computation. This approach significantly optimizes performance and resource utilization.

#### Common Mistakes
Not using a distributed cache can lead to redundant computations and increased latency.
Improper handling of edge cases where the same text is requested at exactly the same time (race conditions).
Failing to clear or refresh cached embeddings when they become outdated.

