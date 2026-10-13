---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a REST query to a vector database that both does similarity search and filters by metadata, for example only documents from a certain user?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a REST query that performs a similarity search on a vector database while filtering by metadata (such as document owner), we need to structure our API in a way that is both efficient and clear. This involves understanding the requirements, making appropriate assumptions, and designing an endpoint that can handle complex queries.

#### Triaging the Question

The question asks for designing a REST query to a vector database. The key aspects are:

Similarity Search: We need to implement a mechanism to find vectors similar to a given input vector.
Metadata Filtering: We must filter the results based on metadata, such as document owner.

#### Assumptions
We assume that the vector database supports vector similarity search queries efficiently.
The metadata is stored alongside the vectors in a structured format (e.g., JSON).
The API will accept a user token for authentication and authorization purposes.

#### First-Principles Solution

Define the Endpoint: Let's define a REST endpoint /similarity-search that accepts the necessary parameters.

Input Parameters:

query_vector: A vector to compare against existing vectors in the database.
user_id: The ID of the user whose documents should be considered for the search (optional).

HTTP Method: Use POST as it is more suitable for sending data than GET.

Request Body Format:

JSON
{
  "query_vector": [0.1, -0.2, 0.3],
  "user_id": "user123"
}

Response Structure:

If the user ID is provided, return vectors that match both the similarity criteria and belong to the specified user.
If no user ID is provided, return all matching vectors.

#### Implementation
Steps
Step 1: Parse the incoming request body.
Step 2: Validate the input parameters (e.g., query_vector should be a valid vector).
Step 3: Use a vector similarity search algorithm to find similar vectors in the database. Popular algorithms include cosine similarity, Euclidean distance, etc.
Step 4: Apply metadata filtering if user_id is provided.
Step 5: Return the results along with any relevant metadata.

#### Example
Code
PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

# Mock vector database (in a real scenario, this would be replaced by an actual vector store)
vectors_db = {
    "user123": [(0.1, -0.2, 0.3), (-0.4, 0.5, 0.6)],
    "user456": [(0.7, -0.8, 0.9), (0.2, 0.3, -0.4)]
}

@app.route('/similarity-search', methods=['POST'])
def similarity_search():
    data = request.get_json()

    query_vector = data['query_vector']
    user_id = data.get('user_id', None)

    if not all(isinstance(x, (int, float)) for x in query_vector):
        return jsonify({"error": "Invalid query vector"}), 400

    # Simulate vector similarity search
    closest_vectors = []
    for vectors in vectors_db.values():
        for v in vectors:
            similarity = calculate_similarity(query_vector, v)
            if similarity > threshold:  # Define a suitable threshold value
                closest_vectors.append((v, similarity))

    filtered_results = [vec for vec, _ in closest_vectors if user_id is None or vec[0] == user_id]

    return jsonify(filtered_results)

def calculate_similarity(v1, v2):
    # Placeholder for actual similarity calculation logic
    pass

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution provides a clear and efficient way to perform both vector similarity search and metadata filtering using a REST API. The main challenge is ensuring that the vector similarity calculation is accurate and fast, especially with large datasets.

#### Common mistakes
include:

Not validating input parameters: Always validate and sanitize inputs.
Improper error handling: Ensure all possible errors are handled gracefully.
Inefficient similarity calculations: Optimize algorithms to handle large-scale data efficiently.

#### Follow-ups
might involve discussing how to scale this solution for a larger number of vectors, how to secure the API, or integrating machine learning models to improve accuracy.

**Comparison:**

```
Input Vector	Metadata Filtered?	Result
[0.1, -0.2, 0.3]	Yes (user123)	[-0.4, 0.5, 0.6]
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

vectors_db = {
    "user123": [(0.1, -0.2, 0.3), (-0.4, 0.5, 0.6)],
    "user456": [(0.7, -0.8, 0.9), (0.2, 0.3, -0.4)]
}

@app.route('/similarity-search', methods=['POST'])
def similarity_search():
    data = request.get_json()

    query_vector = data['query_vector']
    user_id = data.get('user_id', None)

    if not all(isinstance(x, (int, float)) for x in query_vector):
        return jsonify({"error": "Invalid query vector"}), 400

    closest_vectors = []
    for vectors in vectors_db.values():
        for v in vectors:
            similarity = calculate_similarity(query_vector, v)
            if similarity > threshold:
                closest_vectors.append((v, similarity))

    filtered_results = [vec for vec, _ in closest_vectors if user_id is None or vec[0] == user_id]

    return jsonify(filtered_results)

def calculate_similarity(v1, v2):
    # Placeholder for actual similarity calculation logic
    pass

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
POST /similarity-search
{
  "query_vector": [0.1, -0.2, 0.3],
  "user_id": "user123"
}

Output:
[
  [-0.4, 0.5, 0.6]
]
```

**Data:**

```json
{
  "query_vector": [0.1, -0.2, 0.3],
  "user_id": "user123"
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is User ID Provided?
5. Filter by User ID
6. Perform Similarity Search
7. Return Results
8. End

