---
type: "practice"
difficulty: "Basic"
---

# What does REST stand for and what are its core architectural principles?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

REST stands for Representational State Transfer, a set of design constraints for building web services. At its core, REST is an architectural style that emphasizes stateless communication over a network to achieve scalability and maintainability in distributed systems. In this overview, we'll delve into the meaning behind "REPRESENTATIONAL," "STATE TRANSFER," and the key principles that define RESTful APIs.

First, let's restate what the question is really asking: the interviewer wants you to explain the core concepts of REST and its fundamental architectural principles. We will assume a basic understanding of HTTP protocols and web services.

REPRESENTATIONAL: This principle focuses on representing resources in a standard format. Common formats include JSON and XML, which are used to exchange data between client and server.
STATE TRANSFER: The state is maintained by the client (e.g., browser), not the server. Each request from the client should be self-contained, carrying all necessary information to understand the request or respond with the desired representation.

Next, we will explore these principles step-by-step using first-principles reasoning.

Client-Server Architecture: REST enforces a client-server architecture where the client and server communicate through a well-defined API. The server does not keep any state about the interactions with the client.
Stateless Communication: Each request from the client to the server must contain all the information needed to understand and complete the request. This means no session states should be stored on the server, ensuring scalability and maintainability.
Cacheable Requests: Clients can cache responses if they are tagged as cacheable. This helps in reducing load and improving performance by avoiding unnecessary requests.
Uniform Interface: The interface between clients and servers is standardized using HTTP methods (GET, POST, PUT, DELETE) for CRUD operations. URIs are used to identify resources, and the server responds with representations of those resources.

Now let's summarize and address common mistakes:

REST does not stand for "Random Endpoint Stuff Together."
RESTful APIs should be designed with the above principles in mind.
Stateless communication is crucial; each request must carry all required data.
Cacheable responses help in reducing load but must be used judiciously.

**Comparison:**

```
HTTP Method	Usage
GET	Retrieve a representation of a resource.
POST	Create or update a resource.
PUT	Update a resource.
DELETE	Delete a resource.
```

**Code:**

```python
# Example of a simple GET request using Python's requests library
import requests

response = requests.get('https://api.example.com/resources/123')
print(response.json())
```

**Example input / output:**

```

Given the input:

HTTP
Copy
GET /resources/123 HTTP/1.1
Host: api.example.com
Accept: application/json

The expected output (response):

JSON
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
  "resource": {
    "id": 123,
    "name": "Example Resource",
    "description": "This is an example resource."
  }
}
```

**Data:**

```json
{
  "resource": {
    "id": 123,
    "name": "Example Resource",
    "description": "This is an example resource."
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. Client
5. Is the request cacheable?
6. Does the client need tounderstand the state of theserver?
7. Use HTTP methods for CRUDoperations
8. Store response as a cachedobject
9. Server processes request andreturns a representation of theresource

