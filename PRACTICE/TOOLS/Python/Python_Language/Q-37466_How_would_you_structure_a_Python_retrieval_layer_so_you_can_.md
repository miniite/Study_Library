---
type: "practice"
difficulty: "Deep"
---

# How would you structure a Python retrieval layer so you can swap between dense, sparse (BM25), and hybrid search behind one interface, and how do you fuse the two score distributions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ RAG Grounding Issues](<../../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a flexible retrieval layer in Python that supports both dense and sparse (e.g., BM25) search methods, while allowing hybrid fusion of their scores, we need to structure our code in a modular way that abstracts away the underlying implementation details. This approach ensures that different search strategies can be easily swapped out or extended without changing the client-facing interface.

#### Triage

The core requirement is to create an abstraction layer that decouples the retrieval logic from the specific search methods (dense, sparse, and hybrid). The interviewer is probing your ability to design a flexible system, handle multiple algorithms, and manage complex data structures. We'll assume we have access to pre-processed vectors for dense searches and document-term matrices for BM25.

#### First Principles Solution
Define an Abstract Base Class (ABC) for Retrieval: This will ensure that all search methods implement a common interface.
Implement Specific Search Methods: Create classes for each type of search method, inheriting from the ABC.
Hybrid Fusion Strategy: Develop a mechanism to combine scores from both dense and sparse searches.
Detailed Implementation
Step 1: Define Abstract Base Class
PYTHON
from abc import ABC, abstractmethod

class RetrievalLayer(ABC):
    @abstractmethod
    def search(self, query, k=5):
        pass

Step 2: Implement Specific Search Methods

Dense Search Method

PYTHON
class DenseSearch(RetrievalLayer):
    def __init__(self, vectorizer, index):
        self.vectorizer = vectorizer
        self.index = index

    def search(self, query, k=5):
        # Vectorize the query and retrieve top-k results from the index
        query_vector = self.vectorizer.transform([query])
        scores, indices = self.index.search(query_vector.toarray(), k)
        return [(self.index.doc_id[i], score) for i, score in zip(indices[0], scores[0])]

Sparse Search Method (BM25)

PYTHON
class SparseSearch(RetrievalLayer):
    def __init__(self, document_term_matrix, avg_doc_length, doc_lengths):
        self.document_term_matrix = document_term_matrix
        self.avg_doc_length = avg_doc_length
        self.doc_lengths = doc_lengths

    def search(self, query, k=5):
        # Convert the query to a term vector and calculate BM25 scores
        query_vector = {term: 1 for term in set(query.split())}
        bm25_scores = {}

        for doc_id in range(len(self.doc_lengths)):
            score = 0.0
            for term, freq in query_vector.items():
                if term in self.document_term_matrix[:, doc_id]:
                    df = sum(self.document_term_matrix[term, :].toarray().ravel())
                    idf = log((1 + num_docs) / (1 + df))
                    score += idf * ((freq * (k1 + 1)) / (freq + k1 * (1 - b + b * self.doc_lengths[doc_id] / self.avg_doc_length)))
            bm25_scores[doc_id] = score

        # Sort and return top-k documents
        sorted_docs = sorted(bm25_scores.items(), key=lambda x: x[1], reverse=True)[:k]
        return [(doc_id, score) for doc_id, score in sorted_docs]
Step 3: Hybrid Fusion Strategy
PYTHON
class HybridSearch(RetrievalLayer):
    def __init__(self, dense_searcher, sparse_searcher):
        self.dense_searcher = dense_searcher
        self.sparse_searcher = sparse_searcher

    def search(self, query, k=5):
        dense_results = self.dense_searcher.search(query, k)
        sparse_results = self.sparse_searcher.search(query, k)

        # Merge and rank results based on fused scores
        combined_scores = {doc_id: 0.0 for doc_id in range(len(dense_results))}

        for doc_id, score in dense_results:
            combined_scores[doc_id] += score

        for doc_id, score in sparse_results:
            if doc_id not in combined_scores:
                combined_scores[doc_id] = score
            else:
                combined_scores[doc_id] += score * 0.5  # Adjust the fusion weight as needed

        sorted_docs = sorted(combined_scores.items(), key=lambda x: x[1], reverse=True)[:k]
        return [(doc_id, score) for doc_id, score in sorted_docs]

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that we can easily add new search methods by simply implementing a new class inheriting from RetrievalLayer. The hybrid fusion strategy combines the strengths of both dense and sparse searches. Key considerations include:

Efficiency: Ensure that vectorization, similarity calculations, and sorting are optimized for performance.
Scalability: Manage large document sets efficiently, possibly using distributed systems or indexing techniques.
Flexibility: Allow parameter tuning within each search method to optimize results.

#### Follow-up
questions might include:

How would you handle real-time updates to the index?
What about other hybrid methods (e.g., BM25 + TF-IDF)?
How do you ensure that the fused score is meaningful?

**Comparison:**

```
Document ID	Dense Score	Sparse Score	Fused Score
0	0.8	0.9	1.2
1	0.6	0.7	1.3
```

**Code:**

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from scipy.sparse import csr_matrix

# Example usage
vectorizer = TfidfVectorizer()
index = vectorizer.fit_transform(["example document"])

dense_searcher = DenseSearch(vectorizer, index)
sparse_searcher = SparseSearch(document_term_matrix=csr_matrix([[1, 2], [3, 4]]),
                               avg_doc_length=2.0, doc_lengths=[5, 6])

hybrid_searcher = HybridSearch(dense_searcher=dense_searcher, sparse_searcher=sparse_searcher)

query = "example query"
results = hybrid_searcher.search(query, k=3)
print(results)
```

**Example input / output:**

```text
query = "example query"
results = hybrid_searcher.search(query, k=3)
print(results)  # Output will be a list of document IDs and their fused scores
```

**Data:**

```json
{
    "document_term_matrix": [[1, 2], [3, 4]],
    "avg_doc_length": 2.0,
    "doc_lengths": [5, 6]
}
```

**Flow:**

1. Yes
2. No
3. Query
4. Is Dense Search?
5. DenseSearch.search()
6. SparseSearch.search()
7. C,D
8. FuseScores()
9. Return Top-k Results

