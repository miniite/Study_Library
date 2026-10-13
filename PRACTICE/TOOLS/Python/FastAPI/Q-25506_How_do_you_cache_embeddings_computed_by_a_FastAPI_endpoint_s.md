---
type: "practice"
difficulty: "Deep"
---

# How do you cache embeddings computed by a FastAPI endpoint so repeated identical texts skip re-encoding, and when does that cache become unsafe after a model update?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the caching of embeddings in a FastAPI endpoint, we need to implement an efficient mechanism to detect whether text inputs are identical and skip re-encoding if they match cached results. This involves using a cache system that can handle string keys (the input texts) and store corresponding vector representations. The key challenge is ensuring that the cache remains safe after model updates, as outdated embeddings could lead to incorrect results.

FastAPI provides a flexible and performant framework for creating web APIs, making it an ideal choice for implementing such a feature. To achieve this, we can use Python's caching libraries like functools.lru_cache or a dedicated caching solution like Redis. Here’s how we can approach the problem:

Understand the Cache Mechanism: We need to store embeddings in a cache with keys derived from input texts.
Model Update Handling: The cache must be invalidated when the model updates, ensuring that outdated embeddings are not used.
Concurrency and Data Consistency: Ensure that multiple requests do not interfere with each other or corrupt cache data.

Let's walk through the steps to implement this in a FastAPI application.

