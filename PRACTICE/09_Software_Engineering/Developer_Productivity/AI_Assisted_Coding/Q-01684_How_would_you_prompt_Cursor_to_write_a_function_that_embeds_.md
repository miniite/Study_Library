---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt Cursor to write a function that embeds a user query and retrieves the top-k matching chunks, and what would you check about k?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Prompting for Production Patterns](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question effectively, we need to break down how to interact with an AI-assisted coding tool like Cursor to generate a function that embeds user queries and retrieves top-k matching chunks from a dataset. This involves understanding what k represents in the context of results retrieval and ensuring our prompt is clear and precise.

#### Triaging the Question

The question asks for a detailed plan on how to interact with an AI tool (Cursor) to write a specific function. The core of the task is defining the function's behavior, which involves embedding user queries and retrieving top-k matching chunks from a dataset. Here, k represents the number of results we want to retrieve.

Key Assumptions
Dataset: We have a predefined dataset of text or documents.
Embedding: We use an existing model or service that can generate embeddings for both user queries and dataset chunks.
Cursor Capabilities: Cursor is capable of executing the written function with appropriate API calls.

#### First-Principles Solution

Define Function Name and Parameters:

Function name: get_top_k_matches(query, k)
Parameters:
query: The user's query string.
k: An integer specifying how many top matches to return.

Embed Queries and Data Chunks:

Use the embedding model to get vectors for both the user query and each data chunk in the dataset.

Compute Similarities:

Compute cosine similarity between the user query vector and each dataset vector.

Sort and Retrieve Top-k Results:

Sort results based on computed similarities.
Return top k matches along with their corresponding original text chunks or metadata.
Detailed Steps

Prompt Cursor to Define the Function:

PLAINTEXT
Write a Python function called 'get_top_k_matches(query, k)' that takes in a user query and an integer k. The function should return the top-k similar chunks from a dataset of documents based on cosine similarity.

Embedding Process:

Use the embedding model to generate vector representations for both the query and each document chunk.
PYTHON
def get_embedding(text):
    # Placeholder for actual embedding logic
    return np.array([0.1, 0.2, 0.3])

query_vector = get_embedding(user_query)
document_vectors = [get_embedding(chunk) for chunk in dataset_chunks]

Similarity Calculation:

Calculate cosine similarity between the user query vector and each document vector.
PYTHON
from sklearn.metrics.pairwise import cosine_similarity

similarities = [cosine_similarity(query_vector.reshape(1, -1), doc_vector.reshape(1, -1)) for doc_vector in document_vectors]

Sort and Select Top-k Matches:

Sort the documents based on their similarity scores.
PYTHON
sorted_indices = np.argsort(similarities)[::-1]  # Descending order
top_k_indices = sorted_indices[:k]
top_k_chunks = [dataset_chunks[i] for i in top_k_indices]

Return the Results:

Return a list of top-k chunks or their metadata.
PYTHON
def get_top_k_matches(user_query, k):
    query_vector = get_embedding(user_query)
    document_vectors = [get_embedding(chunk) for chunk in dataset_chunks]

    similarities = cosine_similarity(query_vector.reshape(1, -1), np.array(document_vectors))
    sorted_indices = np.argsort(similarities)[::-1]  # Descending order
    top_k_indices = sorted_indices[:k]
    return [dataset_chunks[i] for i in top_k_indices]

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Misunderstanding the role of k (choosing an inappropriate value).
Ignoring edge cases such as when there are fewer than k matches.
Failing to ensure that embeddings are properly normalized.

#### Follow-Up
Questions:

How would you handle situations where k is very large?
What kind of metrics would you use to evaluate the quality of the results?

**Comparison:**

```
Index	Document Chunk	Similarity Score
0	Introduction to machine learning	0.85
1	Understanding neural networks	0.72
2	Machine learning algorithms explained	0.69
3	Application of ML in real-world problems	0.58
```

**Code:**

```python
from sklearn.metrics.pairwise import cosine_similarity

def get_embedding(text):
    # Placeholder for actual embedding logic
    return np.array([0.1, 0.2, 0.3])

def get_top_k_matches(user_query, k):
    query_vector = get_embedding(user_query)
    document_vectors = [get_embedding(chunk) for chunk in dataset_chunks]

    similarities = cosine_similarity(query_vector.reshape(1, -1), np.array(document_vectors))
    sorted_indices = np.argsort(similarities)[::-1]  # Descending order
    top_k_indices = sorted_indices[:k]
    return [dataset_chunks[i] for i in top_k_indices]
```

**Example input / output:**

```text
# Sample input
user_query = "How to use machine learning?"
k = 3

# Expected output (hypothetical values)
top_k_matches = [
    "Introduction to machine learning",
    "Understanding neural networks",
    "Machine learning algorithms explained"
]
```

**Data:**

```json
{
  "user_query": "How to use machine learning?",
  "dataset_chunks": [
    "Introduction to machine learning",
    "Understanding neural networks",
    "Machine learning algorithms explained",
    "Application of ML in real-world problems"
  ]
}
```

**Flow:**

1. Start
2. Define Functionget_top_k_matches(query, k)
3. Embed Query and Documents
4. Compute Cosine Similarities
5. Sort by Similarity Scores
6. Select Top-k Matches
7. Return Results
8. End

