---
type: "practice"
difficulty: "Basic"
---

# What is top-k in retrieval and what does the k value control?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Query Rewriting and Fuzzy Retrieval](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Query_Rewriting_and_Fuzzy_Retrieval.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Top-k retrieval is a common technique used in information retrieval systems, particularly in contexts where you need to fetch a limited number of relevant items from a large dataset. In this approach, "k" represents the number of top-scoring results that are returned. The choice of k value significantly influences how many potential matches will be retrieved and considered for use.

The question is essentially asking about the concept of top-k retrieval and what exactly the parameter k controls in this context.

**Comparison:**

```
k	Retrieved Documents
1	Document #1
2	Document #1, #2
3	Document #1, #2, #3
5+	All documents
```

**Code:**

```python
# Example code to illustrate top-k retrieval (hypothetical)
def retrieve_top_k(query, documents, k):
    # This function would rank the documents based on their relevance to the query
    ranked_documents = rank_documents(query, documents)
    return [doc for _, doc in sorted(ranked_documents.items(), key=lambda item: item[0], reverse=True)[:k]]
```

**Example input / output:**

```

Given the documents above and a query like "machine learning," retrieving top-2 results might return:

PLAINTEXT
Copy
Top-2 Documents for Query "machine learning":
1. "A document about machine learning basics"
2. "Understanding natural language processing" (assuming NLP concepts are somewhat related)
```

**Data:**

```json
{
  "documents": [
    {"id": 1, "content": "A document about machine learning basics"},
    {"id": 2, "content": "An advanced guide to deep learning techniques"},
    {"id": 3, "content": "Understanding natural language processing"},
    {"id": 4, "content": "Introduction to reinforcement learning"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Input Query
4. Is k > number of relevantdocs?
5. Rank all docs
6. Return top-k results
7. Return all relevant docs

