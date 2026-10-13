---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between dense retrieval and keyword (sparse) retrieval?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In information retrieval, the choice between dense retrieval and keyword (sparse) retrieval can significantly impact how well a system performs in tasks such as document search or question answering. Dense retrieval involves using dense vector embeddings to represent text and then finding similar vectors to answer queries, whereas sparse retrieval relies on traditional keyword matching techniques.

#### Triaging the Question

The core of this question is understanding the fundamental differences between these two retrieval strategies. The interviewer might be probing your knowledge of how these methods work under the hood, their pros and cons, and when they are best applied. Key assumptions include familiarity with vector embeddings in dense retrieval and index-based keyword matching in sparse retrieval.

Dense Retrieval vs. Keyword (Sparse) Retrieval
Dense Retrieval:
Definition: This method uses deep learning models to generate fixed-length dense vectors for each piece of text, allowing the system to find semantically similar documents based on vector similarity.
Process:
Embedding Generation: Text is fed into a pre-trained model (e.g., BERT) that outputs dense vectors.
Query Embedding: The query is also transformed into a vector.
Similarity Calculation: Cosine similarity or another metric is used to find the most similar documents.

#### Pros
Higher precision and recall due to semantic understanding.
Better handling of synonyms, paraphrases, and more complex queries.

#### Cons
Computationally intensive during embedding generation.
Requires significant storage for large vocabularies.
Keyword (Sparse) Retrieval:
Definition: This method relies on traditional techniques like TF-IDF or BM25 to match keywords in the query against a collection of documents stored as inverted indexes.
Process:
Inverted Index Construction: Documents are indexed based on their words, creating an index that maps each word to its occurrences across documents.
Query Parsing: The query is broken down into individual terms (keywords).
Relevance Scoring: Documents containing the keywords are scored and ranked.

#### Pros
Efficient in terms of storage and computation for large-scale retrieval systems.
Faster response times due to simpler algorithms.

#### Cons
Lower accuracy on complex queries involving synonyms or context.
Less effective at handling novel or rare words.

#### First Principles Solution

To truly understand the difference, let's walk through a basic example. Consider a search query: "best practices in data science."

Dense Retrieval Example:
Embedding Generation:
The query is fed into BERT to generate its vector representation.
Document Embeddings:
Each document about data science best practices has a pre-computed dense embedding.
Similarity Calculation:
Cosine similarity between the query vector and document vectors is computed.
Top K Documents:
The top K most similar documents are returned.
Keyword (Sparse) Retrieval Example:
Inverted Index Construction:
An inverted index maps "best," "practices," "data," and "science" to their respective document IDs.
Query Parsing:
The query is split into keywords: ["best", "practices", "data", "science"].
Relevance Scoring:
Documents containing all or a majority of these words are scored.
Top K Documents:
The top K most relevant documents are returned based on keyword presence and frequency.

#### Closing Summary

In summary, dense retrieval is more powerful for complex queries due to semantic understanding but requires significant resources, whereas sparse retrieval is faster and more efficient in storage but less accurate for nuanced inquiries. The choice depends heavily on the specific use case and resource constraints.

#### Common Mistakes
and Follow-Ups
Mistake: Overlooking the computational overhead of dense models.

#### Follow-Up
Discussing the trade-offs between precision, recall, and computational efficiency.

**Comparison:**

```
Dense Retrieval	Keyword Retrieval
Uses dense vectors for semantic understanding	Relys on keywords and inverted indexes
Higher computational cost due to model inference	Lower computational cost due to simple algorithms
Better handling of complex queries	Less effective for complex queries
```

**Code:**

```python
# Example using Hugging Face's Sentence Transformers for dense retrieval
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
query_embedding = model.encode("best practices in data science")
document_embeddings = model.encode(["Document 1", "Document 2", ...]) # Pre-computed embeddings

# Using cosine similarity to find top K documents
from sklearn.metrics.pairwise import cosine_similarity
similarities = cosine_similarity(query_embedding.reshape(1, -1), document_embeddings)
top_k_indices = similarities.argsort()[0][-k:]
```

**Example input / output:**

```
Input:
Query: best practices in data science
Documents:
JSON
Copy
[
  {"id": 1, "text": "Document on data science best practices"},
  {"id": 2, "text": "Another document about data practices"}
]
Output (Top K documents based on dense retrieval):
JSON
Copy
[{"id": 1, "score": 0.85}, {"id": 2, "score": 0.65}]
```

**Data:**

```json
{
  "query": "best practices in data science",
  "documents": [
    {"id": 1, "text": "Document on data science best practices"},
    {"id": 2, "text": "Another document about data practices"}
  ]
}
```

**Flow:**

1. Query
2. Embedding Generation
3. Dense Vector
4. Document Embeddings
5. Similarity Calculation
6. Top K Documents
7. Query
8. Keyword Parsing
9. Term Occurrences
10. Relevance Scoring
11. Top K Documents

