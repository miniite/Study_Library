---
type: "practice"
difficulty: "Basic"
---

# How do vector databases like Pinecone or Qdrant expose their functionality over REST? Name the kinds of operations you would call.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with vector databases such as Pinecone or Qdrant, understanding how they expose their functionality through REST APIs is crucial. REST (Representational State Transfer) APIs are a popular choice due to their simplicity and wide support across various platforms and languages. In this context, we'll focus on the basic operations you can perform using these APIs.

#### Triage

The question essentially asks about the types of operations that vector databases like Pinecone or Qdrant expose through their REST API. We need to identify common actions such as creating collections, adding vectors, querying vectors, and managing data.

#### First-Principles Solution
Collection Management: Operations to create, list, update, and delete collections.
Vector Addition/Insertion: Methods for inserting new vectors into a collection.
Vector Queries: APIs that allow performing similarity searches using queries like "find top N nearest neighbors."
Metadata Handling: Managing metadata associated with vectors or collections.

#### Common Mistakes/Follow-Ups
Incorrectly assuming that all operations are available via the REST API (e.g., not all low-level operations might be exposed).
Overlooking the importance of authentication and authorization in securing your API.
Ignoring the need for rate limiting to prevent abuse.

#### Closing Summary

In summary, vector databases like Pinecone or Qdrant expose a range of REST-based operations to manage collections, add vectors, perform queries, and handle metadata. Understanding these operations is essential for effectively working with such systems in real-world applications.

**Code:**

```python
# Example POST request to add a vector (using Python requests library)
import requests

url = "https://api.pinecone.io/v1/gc/project/<project_name>/index/<index_name>/vectors"
headers = {
    'Content-Type': 'application/json',
    'Api-Key': '<your_api_key>'
}
data = [
    {"vector": [0.2, 0.3, -0.5], "id": "vec1"},
    {"vector": [-0.1, 0.8, 0.6], "id": "vec2"}
]

response = requests.post(url, headers=headers, json=data)
print(response.json())
```

**Example input / output:**

```text
Input:
POST https://api.pinecone.io/v1/gc/project/<project_name>/index/<index_name>/vectors

Headers:
Content-Type: application/json
Api-Key: <your_api_key>

Data:
[
    {"vector": [0.2, 0.3, -0.5], "id": "vec1"},
    {"vector": [-0.1, 0.8, 0.6], "id": "vec2"}
]

Output:
{
    "insert_response": {
        "added_vectors": [
            { "id": "vec1", "index": 0 },
            { "id": "vec2", "index": 1 }
        ],
        "upserted_ids": ["vec1", "vec2"]
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. Yes
6. No
7. Create Collection
8. Collection Exists?
9. List Collections
10. Create Collection
11. Collection Created?
12. Insert Vectors
13. Vectors Inserted?
14. Query Vectors
15. Results Retrieved?
16. Return Results
17. Error Handling

