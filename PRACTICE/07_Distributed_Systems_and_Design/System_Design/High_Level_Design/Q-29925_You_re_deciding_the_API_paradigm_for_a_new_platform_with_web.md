---
type: "practice"
difficulty: "Deep"
---

# You're deciding the API paradigm for a new platform with web mobile and partner consumers — architect a strategy that mixes GraphQL for product surfaces and gRPC internally and defend where each boundary sits and how you avoid duplicating auth and validation.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API strategy for a new platform with web, mobile, and partner consumers, mixing GraphQL for public product surfaces and gRPC internally is a strategic choice that leverages the strengths of each protocol while ensuring consistency in authentication and validation. The key decision points involve understanding the trade-offs between these protocols and positioning them appropriately.

Firstly, GraphQL is chosen for its powerful query flexibility and strong type-safety capabilities, making it an excellent fit for public-facing APIs where clients need to retrieve specific data based on complex queries. On the other hand, gRPC excels in internal service communication due to its binary format, high performance, and ability to support multiple languages out of the box.

To ensure consistency and avoid duplicating authentication and validation logic, we need a robust authentication strategy that can be seamlessly applied across both GraphQL and gRPC boundaries. This involves leveraging OAuth 2.0 for authorization and implementing JWT tokens for secure user sessions.

Here's how we would architect this:

External API Layer (GraphQL): This layer will handle all external requests from web, mobile, and partner consumers. It will provide a flexible query interface that allows clients to fetch data in a structured manner.
Internal Service Communication (gRPC): This layer will handle internal service-to-service communication, ensuring high performance and efficient data transfer between different microservices.

#### Triage
What the Question is Really Asking

The core of this question revolves around:

Choosing the right protocol for each API boundary based on its use case.
Ensuring consistent authentication and validation mechanisms across both GraphQL and gRPC boundaries to avoid redundancy.
Designing a robust architecture that balances performance, flexibility, and security.

#### First Principles Solution
External API Layer (GraphQL)
External Requests: Clients will send requests to the GraphQL endpoint with specific query parameters.
Authentication: Validate JWT tokens in incoming requests using an authentication service.
Query Handling: Parse GraphQL queries and translate them into internal operations, ensuring type-safety and data integrity.
Data Aggregation: Fetch necessary data from various microservices (using gRPC) and return the structured response to the client.
Internal Service Communication (gRPC)
Service-to-Service Requests: Microservices will communicate with each other using gRPC for high-performance internal operations.
Authentication: Use mutual TLS (mTLS) for secure service-to-service communication, ensuring only authorized services can access each other.
Validation: Implement validation logic at the microservice level to ensure data consistency and integrity.

#### Closing Summary

By architecting the platform with GraphQL for public product surfaces and gRPC internally, we leverage the strengths of both protocols while maintaining a consistent authentication strategy across all boundaries. This ensures high performance, security, and maintainability in our API design.

#### Common Mistakes/Follow-Ups
Mistake: Failing to implement robust authentication mechanisms that can be reused across GraphQL and gRPC.

#### Follow-Up
How would you handle rate limiting and caching for the external GraphQL API?

**Comparison:**

| Protocol | Use Case            | Authentication | Validation |
|----------|---------------------|----------------|------------|
| GraphQL  | Public product APIs | JWT tokens     | Type-safe  |
| gRPC     | Internal services   | mTLS           | In service |

**Code:**

```
Example of validating JWT tokens in a GraphQL resolver

def validate_token(token): try: jwt.decode(token, options={"verify_signature": False}) return True except Exception as e: print(e) return False

def handle_graphql_request(query): if not validate_token(request.headers.get("Authorization")): raise AuthenticationError("Invalid token")

CODE
Copy
# Parse and execute the GraphQL query
result = execute_graphql(query)

return result
CODE
Copy

```python
# Example of a gRPC service for internal communication
class InternalService(grpc.ServiceServicer):
    def __init__(self, config):
        self.config = config

    @grpc.maybe_stream
    async def FetchData(self, request, context):
        try:
            # Perform data fetch using internal logic
            data = await fetch_data_from_source(request.id)
            yield data
        except Exception as e:
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details(str(e))
```

**Example input / output:**

```text
Input (GraphQL):
POST /graphql
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

{
  "query": "query { product(id: 1) { name, price } }"
}

Output:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "product": {
      "name": "Sample Product",
      "price": 99.99
    }
  }
}
```

**Data:**

```json
{
  "graphql": {
    "endpoint": "/graphql",
    "auth_service": "auth-service:8080"
  },
  "grpc": {
    "services": [
      { "name": "internal-service", "address": "service-internal:50051" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Client
4. Is Authenticated?
5. Fetch Data via GraphQL
6. Authentication Error
7. Query Execution
8. Data Response

