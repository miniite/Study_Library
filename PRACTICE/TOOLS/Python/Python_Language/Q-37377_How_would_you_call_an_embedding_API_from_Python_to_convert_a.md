---
type: "practice"
difficulty: "Intermediate"
---

# How would you call an embedding API from Python to convert a list of documents into vectors, and what shape does the result have?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Token Budgeting for RAG](<../../../../TOOLS/Python/Python_Language/Token_Budgeting_for_RAG.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To call an embedding API from Python to convert a list of documents into vectors, we need to send HTTP requests to the API endpoint with the document data. The resulting embeddings will be in a specific shape that depends on the API's implementation. Let’s break down the process step-by-step.

First, let's consider the core idea: We'll use Python's requests library to make HTTP POST requests to an embedding API. Each request will contain our list of documents as input data, and we expect vector outputs in return.

