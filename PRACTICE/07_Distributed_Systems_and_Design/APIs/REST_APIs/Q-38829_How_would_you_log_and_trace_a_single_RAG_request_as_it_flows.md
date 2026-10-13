---
type: "practice"
difficulty: "Intermediate"
---

# How would you log and trace a single RAG request as it flows through your embedding, vector search, and LLM REST calls?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To log and trace a single Request-Answer-Generation (RAG) request flowing through various components—embedding, vector search, and Language Model (LLM) REST calls—we need to ensure that each component captures detailed information about the request's journey. This involves setting up a structured logging mechanism that can be queried and analyzed later for debugging or auditing purposes.

The key idea is to embed unique identifiers in the requests and responses at every step of the pipeline, ensuring these identifiers are propagated correctly. This way, we can track the request across different services and identify any bottlenecks or issues efficiently.

**Comparison:**

```
Step	Component	Action	Data Logging
1	Embedding Service	POST /embeddings	Request ID
2	Vector Search	GET /search	Request ID, Response Details
3	LLM Service	POST /generate	Request ID, Response Details
```

**Code:**

```python
import uuid

# Function to generate a unique request ID
def get_request_id():
    return str(uuid.uuid4())

# Example logging function for REST API calls
def log_api_call(request_id, service_name, method, url, response):
    print(f"Request ID: {request_id}, Service: {service_name}, Method: {method}, URL: {url}")
    if response:
        print(f"Response Status Code: {response.status_code}")
```

**Example input / output:**

```text
# Example of a request being logged before it reaches the embedding service
log_api_call("0001-02-03-456789abcdef", "Embedding Service", "POST", "/embeddings", None)

# Example response logging after the embedding service
response = {"embedding_vector": [0.1, 0.2, 0.3], "metadata": {"request_id": "0001-02-03-456789abcdef"}}
log_api_call("0001-02-03-456789abcdef", "Embedding Service", "POST", "/embeddings", response)

# Example of a request being logged before it reaches the vector search service
log_api_call("0001-02-03-456789abcdef", "Vector Search Service", "GET", "/search?vector=[0.1, 0.2, 0.3]", None)

# Example response logging after the vector search service
response = {"search_results": [{"similarity_score": 0.8, "document_id": "doc-1234"}], "metadata": {"request_id": "0001-02-03-456789abcdef"}}
log_api_call("0001-02-03-456789abcdef", "Vector Search Service", "GET", "/search?vector=[0.1, 0.2, 0.3]", response)

# Example of a request being logged before it reaches the LLM service
log_api_call("0001-02-03-456789abcdef", "LLM Service", "POST", "/generate?embedding=[0.1, 0.2, 0.3]&query=Example query text for LLM.", None)

# Example response logging after the LLM service
response = {"generated_text": "Here is the generated answer from the model!", "metadata": {"request_id": "0001-02-03-456789abcdef"}}
log_api_call("0001-02-03-456789abcdef", "LLM Service", "POST", "/generate?embedding=[0.1, 0.2, 0.3]&query=Example query text for LLM.", response)
```

**Data:**

```json
{
  "embedding_request": {
    "request_id": "0001-02-03-456789abcdef",
    "input_text": "Example input text for embedding."
  },
  "search_request": {
    "request_id": "0001-02-03-456789abcdef",
    "embedding_vector": [0.1, 0.2, 0.3],
    "query_text": "Example query text for vector search."
  },
  "llm_request": {
    "request_id": "0001-02-03-456789abcdef",
    "input_embedding_vector": [0.1, 0.2, 0.3],
    "query_text": "Example query text for LLM."
  }
}
```

**Flow:**

1. Embedding Request
2. POST /embeddings
3. Vector Search Request
4. GET /search
5. LLM Request
6. POST /generate

