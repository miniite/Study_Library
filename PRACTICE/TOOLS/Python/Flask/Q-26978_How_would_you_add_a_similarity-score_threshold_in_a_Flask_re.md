---
type: "practice"
difficulty: "Deep"
---

# How would you add a similarity-score threshold in a Flask retrieval endpoint so low-relevance chunks are dropped before prompting?

**Difficulty:** Deep · **Levels:** Fresher · [↑ RAG Endpoints in Flask](<../../../../TOOLS/Python/Flask/RAG_Endpoints_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of adding a similarity-score threshold in a Flask retrieval endpoint, we need to filter out low-relevance chunks based on their similarity score. This involves understanding how the similarity score is calculated, integrating it into our Flask application, and ensuring that only high-similarity results are passed to the next stage (in this case, prompting). Here’s a step-by-step approach:

Understand the Similarity Score: First, we need to understand what the similarity score represents in the context of your retrieval system. This could be based on TF-IDF, cosine similarity, or any other metric that quantifies how similar two pieces of text are.

Define the Threshold: Set a threshold value for the similarity score. Any chunk below this threshold will be considered low-relevance and should not proceed to the prompting stage.

Integrate into Flask Endpoint: Modify your Flask endpoint to include logic for filtering based on the similarity score before processing further.

Testing and Validation: Ensure the endpoint works as expected by testing it with various input data points.

Let's break this down in more detail.

**Code:**

```python
from flask import Flask, request, jsonify
import numpy as np

app = Flask(__name__)

# Example similarity score threshold (let’s set it to 0.6 for demonstration)
SIMILARITY_THRESHOLD = 0.6

def filter_low_relevance_chunks(query_embedding, chunk_embeddings):
    """
    Filters chunks based on a similarity threshold.

    :param query_embedding: Embedding vector of the query
    :param chunk_embeddings: List of embedding vectors for each chunk
    :return: Filtered list of high-relevance chunk embeddings
    """
    similarities = np.dot(chunk_embeddings, query_embedding) / (np.linalg.norm(chunk_embeddings, axis=1) * np.linalg.norm(query_embedding))
    relevant_chunks_indices = [i for i, similarity in enumerate(similarities) if similarity >= SIMILARITY_THRESHOLD]

    return [chunk_embeddings[i] for i in relevant_chunks_indices]

@app.route('/search', methods=['POST'])
def search():
    """
    Endpoint to handle the retrieval request.
    """
    data = request.get_json()
    query = data['query']
    chunk_embeddings = data['chunk_embeddings']

    # Step 1: Embed the query
    from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
    vectorizer = TfidfVectorizer(stop_words='english', max_features=500)
    query_embedding = vectorizer.fit_transform([query]).toarray()

    # Step 2: Filter chunks based on similarity score
    filtered_embeddings = filter_low_relevance_chunks(query_embedding, chunk_embeddings)

    # Step 3: Proceed with prompting using the high-relevance chunks
    # (This part would involve further logic depending on your application)

    return jsonify({"filtered_embeddings": filtered_embeddings.tolist()})

if __name__ == '__main__':
    app.run(debug=True)
```

**Data:**

```json
{
  "query": "How to use Flask for similarity-based search?",
  "chunk_embeddings": [
    [0.1, 0.2, -0.3, 0.4],
    [-0.5, 0.6, 0.7, -0.8],
    [0.9, -1.0, 1.1, -1.2]
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Search Request
4. Query Embedding?
5. Calculate Similarity Scores
6. Filter Low-Relevance Chunks
7. Proceed with Prompting
8. Return No Relevant Results

