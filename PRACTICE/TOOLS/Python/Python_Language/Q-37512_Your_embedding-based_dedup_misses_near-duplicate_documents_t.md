---
type: "practice"
difficulty: "Deep"
---

# Your embedding-based dedup misses near-duplicate documents that hurt retrieval diversity. How would you implement MMR or a diversity-aware re-ranking in Python to reduce redundant chunks in the context?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ RAG Grounding Issues](<../../../../TOOLS/Python/Python_Language/RAG_Grounding_Issues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of embedding-based deduplication missing near-duplicate documents, we can implement a method known as Max Margin Ranking (MMR) or a diversity-aware re-ranking mechanism to ensure that retrieval results are more diverse and representative. This involves ranking documents not just based on their relevance scores but also considering their similarity to already selected documents. In Python, this can be implemented using various libraries such as numpy for numerical operations and faiss for efficient vector search.

#### Triaging the Question

The core of the question is about enhancing document retrieval by reducing redundancy in a ranked list of results. Specifically, we need to ensure that highly similar documents are not selected together, thereby improving the diversity of the final set of documents returned to the user. This involves understanding how MMR works and implementing it in Python.

Key Assumptions
We have a pre-existing ranking mechanism based on embeddings (e.g., BERT, Sentence Transformers).
We need to ensure that selected documents are not too similar to each other.
The goal is to maintain a high overall relevance while ensuring diversity.

#### First-Principles Solution
Document Embeddings: Start by obtaining embeddings for all relevant documents using a pre-trained model like BERT or Sentence Transformers.
Initial Ranking: Rank the documents based on their relevance scores, which can be derived from the cosine similarity of their embeddings to the query vector.
MMR Objective Function: Formulate an objective function that combines both relevance and diversity. The MMR objective aims to maximize the sum of relevance scores while minimizing the pairwise document similarities.
Greedy Selection: Use a greedy algorithm to iteratively select documents that best balance relevance and diversity.
Step-by-Step Implementation

Embedding Generation:

PYTHON
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer('all-MiniLM-L6-v2')
query_embedding = model.encode(query)
document_embeddings = [model.encode(doc) for doc in documents]

Initial Ranking:

PYTHON
relevance_scores = util.cos_sim(query_embedding, document_embeddings).flatten()
ranked_indices = sorted(range(len(relevance_scores)), key=lambda i: relevance_scores[i], reverse=True)

MMR Objective Function: The MMR objective function can be defined as: [ \text{MMR}(\mathbf{s}) = \sum_{i=1}^{k} s_i - \lambda \sum_{1 \leq i < j \leq k} d(i, j) ] where (s_i) is the relevance score of document (i), and (d(i, j)) is a measure of similarity between documents.

Greedy Selection:

PYTHON
def mmr(documents, query_embedding, top_k=5):
    embedding = model.encode
    relevance_scores = util.cos_sim(query_embedding, embedding(document_embeddings)).flatten()
    ranked_indices = sorted(range(len(relevance_scores)), key=lambda i: relevance_scores[i], reverse=True)

    selected_docs = []
    remaining_docs = set(range(len(documents)))

    for idx in ranked_indices:
        if not remaining_docs:
            break
        doc_idx = idx
        if all(dissimilarity(doc_idx, selected_docs) > 0.5 for selected_doc in selected_docs):
            selected_docs.append(doc_idx)
            remaining_docs.remove(doc_idx)

    return [documents[idx] for idx in selected_docs]

def dissimilarity(doc1, docs2):
    # Calculate pairwise similarity between doc1 and all other selected documents
    similarities = util.cos_sim(embedding(documents[doc1]), embedding([documents[doc2] for doc2 in docs2]))
    return min(similarities)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, implementing MMR or a diversity-aware re-ranking involves generating embeddings, ranking documents based on relevance, and then iteratively selecting the most relevant yet diverse documents. Key challenges include choosing an appropriate similarity measure and balancing between relevance and diversity. Common mistakes include not considering the context properly, overfitting to the training data, or ignoring edge cases where documents are very similar.

For follow-ups, one might consider more sophisticated methods like MMR with a soft margin, integrating user feedback loops for dynamic re-ranking, or scaling the solution to handle large document sets efficiently using techniques like vector quantization and distributed computing frameworks.

**Comparison:**

```
Document	Relevance Score
It's sunny and warm.	0.95
The temperature will drop tonight.	0.78
Today’s forecast predicts rain.	0.62

Selected Documents:

"It's sunny and warm."
"Today’s forecast predicts rain."

Dissimilarity Matrix (Example):

Document	Doc1	Doc2
It's sunny and warm.	-	0.34
The temperature will drop tonight.	0.34	-
```

**Code:**

```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer('all-MiniLM-L6-v2')
query_embedding = model.encode("What is the weather today?")
document_embeddings = [model.encode(doc) for doc in ["It's sunny and warm.", "The temperature will drop tonight.", "Today’s forecast predicts rain."]]

relevance_scores = util.cos_sim(query_embedding, document_embeddings).flatten()
ranked_indices = sorted(range(len(relevance_scores)), key=lambda i: relevance_scores[i], reverse=True)

print("Ranked Documents:", [documents[idx] for idx in ranked_indices])
```

**Example input / output:**

```text
Input:
- Query: "What is the weather today?"
- Pre-trained model: `all-MiniLM-L6-v2`

Output:
- Ranked Documents: `["It's sunny and warm.", "The temperature will drop tonight.", "Today’s forecast predicts rain."]`

Initial Relevance Scores (Cosine Similarity):
- 0.95: "It's sunny and warm."
- 0.78: "The temperature will drop tonight."
- 0.62: "Today’s forecast predicts rain."

Selected Documents after MMR:
- ["It's sunny and warm.", "Today’s forecast predicts rain."]
```

**Data:**

```json
{
  "documents": [
    "It's sunny and warm.",
    "The temperature will drop tonight.",
    "Today’s forecast predicts rain."
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Generate Embeddings
5. Rank Documents
6. Select Initial Document
7. Calculate Dissimilarity withSelected Docs
8. Check Similarity Threshold
9. Add to Selection List
10. Iterate Next Document
11. End

