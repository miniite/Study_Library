---
type: "practice"
difficulty: "Expert"
---

# Design the reference architecture for a search platform that must serve lexical, semantic, and hybrid retrieval as first-class products to any team. What abstractions do you expose so teams don't need to understand HNSW or BM25 internals?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a search platform that supports lexical, semantic, and hybrid retrieval, our primary goal is to abstract away the complexity of underlying algorithms like HNSW (Hierarchical Navigable Small World) or BM25 while providing rich features that enable teams to build and deploy search functionality without needing deep understanding of these internals. We will expose a set of high-level abstractions tailored for different types of searches, ensuring ease of use and flexibility.

First, let's break down what the question is probing: we need to design an architecture where various teams can seamlessly integrate searching into their applications or workflows with minimal effort. This involves creating a clear interface that abstracts away the complexity of implementing and tuning search algorithms. We will leverage Elasticsearch’s robust infrastructure but design our API such that users only interact with meaningful, well-defined entities.

#### Assumptions
include:

Teams using this platform are varied in terms of technical expertise.
The system must be scalable to handle large volumes of data and queries efficiently.
There is a need for both real-time and near-real-time search capabilities.

Step-by-step, we will start by defining the core abstractions needed:

Search Queries: Define different types of query objects that encapsulate what users might want to achieve (e.g., exact match, fuzzy match, semantic query).
Document Models: Create a standardized way to represent documents which are indexed and queried.
Search Results: Provide a structured response format for search results including relevance scores, context snippets, etc.

Let’s dive into each abstraction in detail:

Search Queries

We will expose different types of queries that cover the spectrum from simple keyphrase matching to complex semantic searches:

ExactMatchQuery: For precise keyword matching.
FuzzyMatchQuery: For approximate matches with a certain tolerance level.
SemanticQuery: Uses advanced techniques like BERT embeddings for contextually rich search.
Document Models

A flexible schema that can accommodate various types of content while maintaining consistency in how documents are indexed and retrieved:

JSON
{
  "id": "string", // Unique identifier
  "title": "string",
  "content": "string",
  "metadata": {
    "tags": ["tag1", "tag2"], // Metadata tags for categorization
    "timestamp": "datetime"   // Timestamp of when the document was created or updated
  }
}
Search Results

A structured output format that provides not just the matched documents but also contextual snippets, relevance scores, and metadata to aid in further filtering:

JSON
[
  {
    "id": "string",
    "score": number,
    "snippet": string,
    "metadata": {
      "timestamp": "datetime", // When this document was indexed or last updated
      "tags": ["tag1", "tag2"] // Metadata tags for easier categorization
    }
  },
  ...
]

By exposing these abstractions, teams can interact with the search platform using familiar query constructs and receive meaningful results without diving into the intricacies of how Elasticsearch manages its internal data structures or ranking algorithms.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key to success here is ensuring that developers who are not experts in information retrieval still find it easy to implement complex search functionalities. Common pitfalls include overly complicated interfaces or not providing enough detail in the query results. Follow-up questions might revolve around how we handle partial updates, data security, and performance optimization strategies.

**Comparison:**

```
Query Type	Example Usage	Description
ExactMatchQuery	s = s.query("match", content=query_model.content)	Simple keyword match
FuzzyMatchQuery	s = s.query("fuzzy_like_this", content=query_model.content, max_expansions=50)	Approximate matches within a tolerance
SemanticQuery	s = s.query("script_score", query=s.query("match_all"), script="params._score * b['fields']['content'].similarity('exact', params.query)"))	Contextually rich search using BERT embeddings
```

**Code:**

```python
from elasticsearch_dsl import Search

def perform_search(query_type, doc_model):
    s = Search(index="documents")

    if query_type == "exact":
        s = s.query("match", content=query_model.content)
    elif query_type == "fuzzy":
        s = s.query("fuzzy_like_this", content=query_model.content, max_expansions=50)
    elif query_type == "semantic":
        s = s.query("script_score", query=s.query("match_all"), script="params._score * b['fields']['content'].similarity('exact', params.query)")

    response = s.execute()
    return [hit.to_dict() for hit in response.hits]
```

**Example input / output:**

```text
query = SemanticQuery(query="implementing bert embeddings")
results = perform_search("semantic", query)
print(results)
# Example Output:
[
  {
    "id": "doc2",
    "score": 0.85,
    "snippet": "BERT embeddings are used to improve the quality of semantic searches...",
    "metadata": {
      "timestamp": "2023-10-01",
      "tags": ["semantic", "bert"]
    }
  },
  ...
]
```

**Data:**

```json
{
  "documents": [
    {
      "id": "doc1",
      "title": "Introduction to Elasticsearch",
      "content": "Elasticsearch is a search engine based on the Lucene library...",
      "metadata": {
        "tags": ["elasticsearch", "search"],
        "timestamp": "2023-09-01"
      }
    },
    {
      "id": "doc2",
      "title": "Implementing Semantic Search with BERT",
      "content": "BERT embeddings are used to improve the quality of semantic searches...",
      "metadata": {
        "tags": ["semantic", "bert"],
        "timestamp": "2023-10-01"
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Query
6. Query Type Selection
7. Exact Match?
8. ExactMatchQuery
9. Fuzzy Match?
10. FuzzyMatchQuery
11. Semantic Query
12. Use BERT Embeddings forSemantic Search
13. Document Index Search
14. Retrieve Matching Documents
15. Post-processing: ContextSnippets, Relevance Scores
16. Return Results to User

