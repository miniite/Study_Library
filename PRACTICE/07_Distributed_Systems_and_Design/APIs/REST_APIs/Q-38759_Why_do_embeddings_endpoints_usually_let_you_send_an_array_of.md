---
type: "practice"
difficulty: "Intermediate"
---

# Why do embeddings endpoints usually let you send an array of texts in one request instead of one text at a time?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Embeddings APIs often accept multiple text inputs in a single request because this approach is more efficient, scalable, and flexible compared to sending individual requests for each input. This practice leverages batching techniques, which reduce overhead and network latency.

#### Triage
The question asks why embedding endpoints typically allow sending an array of texts in one request rather than handling each text separately. The core idea here is the efficiency gains from batch processing. The interviewer might probe your understanding of HTTP requests, API design principles, and the trade-offs between different approaches.

#### First Principles Solution

Efficiency: Sending multiple texts in a single request reduces the number of round trips to the server. Each individual HTTP request incurs overhead such as TCP handshake and HTTP headers. Reducing these can significantly improve performance, especially when dealing with large volumes of data.

Scalability: Batch processing is inherently more scalable because it processes multiple requests in parallel or sequentially within a single API call. This is particularly important for embedding services that need to handle a high volume of texts simultaneously.

Flexibility and User Experience: Users can send multiple queries at once, which simplifies their workflow. For instance, if you have a dataset with thousands of documents, processing them one by one would be time-consuming and less practical.

Error Handling and Consistency: Batch processing allows for more consistent error handling. If an issue arises during the batch process, it can be handled as a single entity, making debugging easier compared to multiple individual requests.

Rate Limiting and Throttling: APIs often have rate limits or throttling mechanisms. By batching requests, you can better manage these limits without hitting them repeatedly for each input. This is crucial in maintaining service availability and avoiding overloading the server.

Cost Optimization: For cloud services, handling multiple requests at once can lead to cost savings because cloud providers might offer discounts on batch processing or charge based on the total volume of data processed rather than individual queries.

#### Closing Summary
In summary, embedding endpoints typically accept an array of texts in one request to enhance efficiency, scalability, and user experience. This approach reduces network overhead, allows for better error handling, and optimizes resource usage. Understanding these principles is crucial when designing APIs that need to handle large volumes of data efficiently.

#### Common Mistakes/Follow-Ups

Failing to recognize the importance of HTTP request overhead.
Not considering the impact on rate limiting or throttling mechanisms.
Overlooking flexibility in user workflows and error handling.
Ignoring potential cost savings from batch processing, especially in cloud environments.

**Comparison:**

```
Batch Processing	Single Request
Reduced overhead	Higher overhead
Scalable	Less scalable
Consistent error handling	More complex error handling
Simplified user experience	Complicated workflows
Cost optimization	No cost savings
```

**Code:**

```python
# Example Python code for sending a batch of texts to an embedding API
import requests

texts = ["Hello world", "This is a test"]
url = "https://api.example.com/embeddings"

response = requests.post(url, json={"texts": texts})
print(response.json())
```

**Example input / output:**

```text
Request:
POST https://api.example.com/embeddings

Payload:
{
  "texts": [
    "Hello world",
    "This is a test"
  ]
}

Response:
{
  "embeddings": [
    [0.123, -0.456, ...],
    [-0.789, 0.012, ...]
  ]
}
```

**Data:**

```json
{
  "texts": [
    "Hello world",
    "This is a test"
  ]
}
```

**Flow:**

1. User sends batch of texts
2. API receives and processes inparallel
3. Embeddings calculated foreach text
4. Single response with allembeddings returned

