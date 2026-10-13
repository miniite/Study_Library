---
type: "practice"
difficulty: "Intermediate"
---

# How would you retrieve and answer when the knowledge base has no relevant chunk at all?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In scenarios where there is no relevant chunk in a knowledge base, the retrieval strategy should focus on fallback mechanisms that can provide useful information even in the absence of exact matches. This involves leveraging context-aware approaches and external sources to infer or approximate the answer. The key idea is to balance between providing accurate responses and maintaining user satisfaction by offering reasonable alternatives.

#### Triaging the Question

The question asks about handling situations where the knowledge base does not contain any relevant chunks (i.e., pieces of information) that directly match a query. This scenario can occur due to various reasons, such as outdated data, incomplete coverage, or the query being too specific. The interviewer is probing your understanding of retrieval strategies and fallback mechanisms in AI systems.

#### Assumptions
Knowledge Base: A structured database containing relevant information.
Query: An input seeking a piece of information from the knowledge base.
Relevance: A measure indicating how closely a chunk matches the query.

#### First-Principles Solution
1. Context-Aware Approaches:

When no exact match is found, one can use context-aware methods to infer relevant chunks or approximate answers. These methods include:

Semantic Similarity: Measure the similarity between the query and existing chunks using techniques like cosine similarity in vector spaces.
Probabilistic Models: Use probabilistic models to predict which chunks are most likely to be relevant based on historical data.
2. External Data Sources:

Leveraging external data sources can provide additional context or information that may help in formulating a useful response. Common external sources include:

Search Engines: Query search engines like Google or Bing for related information.
APIs: Utilize APIs from databases or services that specialize in the topic of interest.
3. User Feedback and Adaptation:

Implementing user feedback mechanisms can continuously improve the retrieval strategy. For example, if users often ask follow-up questions after receiving an answer, this indicates a need for better context-aware responses.

4. Fallback Strategies:

Develop fallback strategies to provide meaningful responses even when no direct match is found. These strategies might include:

Synonymous Phrases: Use synonyms or related terms that are likely to be relevant.
Default Responses: Predefined responses that acknowledge the lack of information but provide helpful suggestions.
5. Ranking and Prioritization:

Rank and prioritize chunks based on their relevance scores, and present the most probable ones first. This helps in providing useful information even if it’s not a direct match.

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, when dealing with scenarios where no relevant chunk is found, the approach should focus on leveraging context-aware techniques, external data sources, user feedback mechanisms, fallback strategies, and ranking prioritization. Common mistakes include relying solely on exact matches or failing to provide any response at all.

#### Follow-Ups

How would you measure the similarity between a query and chunks in your knowledge base?
Can you explain how external APIs can be integrated into the retrieval process?
What are some potential drawbacks of using fallback strategies, and how do you mitigate them?

**Comparison:**

```
Query	Relevant Chunks
What is the capital of France?	[Chunk 1] The capital of France is Paris.
How many languages are spoken in India?	[] No relevant chunks found.
```

**Code:**

```python
# Example of semantic similarity calculation using cosine similarity
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

def get_relevant_chunks(query):
    vectorizer = TfidfVectorizer()
    tfidf_matrix = vectorizer.fit_transform([query])
    chunks = ["chunk1", "chunk2", "chunk3"]  # Example chunks
    chunk_matrix = vectorizer.transform(chunks)
    similarity_scores = cosine_similarity(tfidf_matrix, chunk_matrix).flatten()
    return chunks[np.argsort(-similarity_scores)[:5]]  # Return top 5 most similar chunks

# Example of using an external search engine API
import requests

def fallback_to_search_engine(query):
    api_key = "YOUR_API_KEY"
    search_url = f"https://api.searchengine.com/v1/search?query={query}&key={api_key}"
    response = requests.get(search_url)
    if response.status_code == 200:
        return response.json()  # Process the JSON data to extract useful information
    else:
        return "Failed to retrieve information from search engine"
```

**Example input / output:**

```text
# Example input and output for fallback mechanism
query = "What is the capital of France?"
relevant_chunks = get_relevant_chunks(query)
print("Top relevant chunks:", relevant_chunks)

if not relevant_chunks:
    result = fallback_to_search_engine(query)
    print(result)
else:
    answer = relevant_chunks[0]["content"]
    print(f"Answer: {answer}")
```

**Data:**

```json
{
  "query": "What is the capital of France?",
  "chunks": [
    {"id": 1, "content": "The capital of France is Paris."},
    {"id": 2, "content": "France has many famous landmarks."}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Query
4. Get Relevant Chunks
5. Find Similarity Scores
6. Provide Top 5 Chunks
7. If no chunks, Fallback toSearch Engine
8. Search Engine API Call
9. Process Response
10. Return Found Information
11. Return Direct Answer fromChunks

