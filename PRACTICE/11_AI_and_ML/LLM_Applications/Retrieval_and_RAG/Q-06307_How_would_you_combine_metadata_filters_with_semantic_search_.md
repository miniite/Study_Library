---
type: "practice"
difficulty: "Intermediate"
---

# How would you combine metadata filters with semantic search, e.g. 'only 2025 policy docs'?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Query Rewriting and Fuzzy Retrieval](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Query_Rewriting_and_Fuzzy_Retrieval.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To combine metadata filters with semantic search effectively, we need to ensure that our system can efficiently retrieve documents based on both structured metadata and unstructured content. The key idea is to leverage a hybrid approach where metadata filters act as initial screening mechanisms, followed by more sophisticated semantic searches to refine the results.

#### Triaging the Question

The question asks us to integrate two methods of information retrieval: metadata filters (e.g., filtering by date) and semantic search (e.g., searching for specific terms within documents). The goal is to implement a system that can return relevant documents based on both criteria. The interviewer might be probing your understanding of query optimization, the relevance of metadata in search engines, and how to balance between simple filters and complex searches.

Step-by-Step Solution
Understand Metadata Filters: Metadata such as document type (e.g., policy), date range (e.g., 2025 policies), author, etc., can be used for initial filtering.
Semantic Search: Once the metadata filters have narrowed down the set of documents, we apply a semantic search to further refine the results based on user queries.
Step-by-Step Implementation

Input Handling:

The user inputs a query with possible filters (e.g., date=2025 AND type=policy).

Metadata Filter Execution:

Apply metadata filters to narrow down the document set.
PYTHON
# Example metadata filtering in Python
filtered_documents = [doc for doc in documents if doc['date'] == 2025 and doc['type'] == 'policy']

Semantic Search within Filtered Documents:

Use a semantic search engine like Elasticsearch or a custom embedding model to find relevant documents based on the user query.
PYTHON
# Example using Elasticsearch for semantic search
search_results = es.search(
    index="documents",
    body={
        "query": {
            "match": {
                "content": query_text
            }
        }
    }
)

Combining Results:

Combine the filtered documents with the results from the semantic search to provide a comprehensive set of relevant documents.
PYTHON
# Example combining results in Python
final_results = list(set(filtered_documents + [hit['_source'] for hit in search_results['hits']['hits']])

Output and Ranking:

Return the combined results, optionally ranked by relevance based on both metadata match and semantic similarity.
PYTHON
# Example output
return final_results[:10]

#### Closing Summary

By combining metadata filters with semantic search, we ensure that our system can efficiently narrow down a large dataset while still providing relevant results. The key is to use metadata for quick filtering and then leverage semantic search to refine the results based on user queries.

#### Common Mistakes
and Follow-Ups

Mistake: Focusing solely on one method (metadata or semantic) without understanding how they complement each other.

#### Follow-Up
Discuss potential issues with over-reliance on metadata filters in cases where data is not well-tagged or when there are complex queries that cannot be easily captured by metadata alone.

Mistake: Not considering performance implications of combining both methods, especially at scale.

#### Follow-Up
Talk about optimizations like caching filtered results and using efficient semantic search algorithms to handle large datasets.

**Comparison:**

```
Document ID	Date	Type	Content
1	2025	policy	Policy details for 2025
2	2024	policy	Policy renewal details for 2024
3	2025	report	Annual report for 2025
```

**Code:**

```python
# Example code for filtering documents based on metadata
filtered_documents = [doc for doc in documents if doc['date'] == 2025 and doc['type'] == 'policy']

# Example Elasticsearch query for semantic search
search_results = es.search(
    index="documents",
    body={
        "query": {
            "match": {
                "content": query_text
            }
        }
    }
)
```

**Example input / output:**

```text
query_text = "policy"
date_filter = 2025
type_filter = 'policy'

# Filter documents based on metadata
filtered_documents = [doc for doc in documents if doc['date'] == date_filter and doc['type'] == type_filter]

# Perform semantic search within filtered documents
search_results = es.search(
    index="documents",
    body={
        "query": {
            "match": {
                "content": query_text
            }
        }
    }
)

final_results = list(set(filtered_documents + [hit['_source'] for hit in search_results['hits']['hits']]))
```

**Data:**

```json
{
  "documents": [
    {"id": 1, "date": 2025, "type": "policy", "content": "Policy details for 2025"},
    {"id": 2, "date": 2024, "type": "policy", "content": "Policy renewal details for 2024"},
    {"id": 3, "date": 2025, "type": "report", "content": "Annual report for 2025"}
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. User Query
5. Are metadata filters specified?
6. Apply Metadata Filters
7. Are there semantic searchterms?
8. Perform Semantic Search
9. Combine Results
10. Directly Apply Semantic Search
11. Return Top 10 Results

