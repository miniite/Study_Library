---
type: "practice"
difficulty: "Deep"
---

# How would you design a FastAPI endpoint that evaluates retrieval quality (recall@k, MRR) against a labelled query set so a vector-index change is caught before it ships?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective FastAPI endpoint for evaluating retrieval quality metrics such as recall@k and Mean Reciprocal Rank (MRR), we need to ensure that our system can efficiently process a query set against a vector index, compute the desired metrics, and provide actionable insights. This involves setting up a robust backend with efficient data access and indexing mechanisms, ensuring scalable performance, and integrating it into a user-friendly API.

The core of this solution is to create an endpoint that:

Accepts a labelled query set.
Queries the vector index for relevant results.
Compares these results against the labels to compute recall@k and MRR.
Returns the metrics along with any necessary context or explanations.

