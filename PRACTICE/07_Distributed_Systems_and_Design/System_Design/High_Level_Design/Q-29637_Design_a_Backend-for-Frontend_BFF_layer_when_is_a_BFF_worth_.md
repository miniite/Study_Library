---
type: "practice"
difficulty: "Intermediate"
---

# Design a Backend-for-Frontend (BFF) layer — when is a BFF worth it, and how do you keep web and mobile BFFs from duplicating logic while shaping payloads per client?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this intermediate-level discussion on designing a Backend-for-Frontend (BFF) layer, we'll explore when such a design is beneficial and how to keep web and mobile BFFs from duplicating logic while customizing payloads per client. We’ll start by defining the BFF’s role in modern microservices architectures, then delve into its advantages and trade-offs. Finally, we will outline strategies for ensuring consistency across different clients.

A BFF acts as a gateway between external services (like APIs) and internal front-end applications, filtering data to match the needs of specific clients. This design is particularly useful in complex microservice architectures where multiple client types interact with diverse sets of backend services.

#### Triage
Understanding the Question

The core question here revolves around determining when to implement a BFF layer and how to manage logic and payload shaping across different client types (web vs. mobile) without duplication. The interviewer is probing your understanding of microservices, API design principles, and the trade-offs involved in such architectural decisions.

#### First-Principles Solution

Identifying When a BFF is Needed:

Complexity: If you have multiple clients (web, mobile, IoT) interacting with common backend services, a BFF can simplify the API surface for each client.
Security: Protecting sensitive data or APIs from direct access by external entities.
Performance Optimization: Caching frequently accessed data to reduce load on backend services and improve response times.

Avoiding Duplication:

Shared Data Models: Use a consistent data model across clients but adapt the API endpoints to client-specific requirements.
API Gateway Patterns: Implement patterns like API Aggregation, where the BFF aggregates data from multiple microservices into a single endpoint tailored for each client.

Customizing Payloads:

Dynamic Query Parameters: Allow dynamic query parameters in APIs to tailor responses based on user roles and permissions.
Client-Specific Endpoints: Design separate endpoints that serve specific needs of clients, like filtering data or altering response formats.
Detailed Steps

Identify Client Requirements:

Define the unique requirements for web and mobile clients, such as screen size, navigation patterns, and data display preferences.

Define BFF Responsibilities:

Create a BFF that acts as an intermediary between microservices and client applications.
Ensure the BFF can handle different types of requests from both web and mobile clients.

Implement API Aggregation:

Web Client
BFF
Mobile Client
BFF
Backend Service 1
Backend Service 2
Backend Service 3
BFF

#### Cons
olidated Response forWeb
BFF

#### Cons
olidated Response forMobile

Customize Responses:

Use middleware in the BFF to filter and transform data based on client type.
Implement conditional logic within the BFF to provide different payloads.

#### Closing Summary

Implementing a BFF layer can significantly improve the efficiency, security, and maintainability of your microservices architecture. By carefully defining client requirements and using API aggregation patterns, you can avoid duplicating logic while shaping payloads tailored for each type of client.

#### Common Mistakes/Follow-Ups

Common Mistake: Overcomplicating the BFF layer with too many customizations, leading to maintenance issues.

#### Follow-Up
Simplify by adhering to a common data model and using dynamic query parameters where necessary.

Common Mistake: Ignoring performance optimizations in the BFF.

#### Follow-Up
Implement caching mechanisms and profile API response times to identify bottlenecks.

**Comparison:**

```
Client Type	Customization Example
Web	Use responsive design for different screen sizes; optimize images and videos.
Mobile	Implement push notifications; optimize for low bandwidth; reduce data usage by compressing content.
```

**Code:**

```python
# Example of a simple BFF middleware for filtering data based on client type
def bff_middleware(request):
    if request.client_type == 'mobile':
        # Custom logic for mobile clients
        return transform_response_for_mobile(api_call())
    elif request.client_type == 'web':
        # Custom logic for web clients
        return transform_response_for_web(api_call())
    else:
        raise ValueError("Unknown client type")
```

**Example input / output:**

```text
Input:
Client Type: mobile
Request Data:
- user_id: 1234
- filters: ["recent", "top_rated"]

Response (Transformed for Mobile Client):
[
    {"title": "Product A (New)", "price": 10.99, "image": "/path/to/image.png"},
    {"title": "Product B (Top Rated)", "price": 5.99}
]
```

**Data:**

```json
{
  "client_type": "mobile",
  "request_data": {
    "user_id": 1234,
    "filters": ["recent", "top_rated"]
  },
  "response_data": [
    {"title": "Product A", "price": 10.99},
    {"title": "Product B", "price": 5.99}
  ]
}
```

**Flow:**

1. Client Requests Data
2. BFF Receives Request
3. Filter/Transform Data Based onClient Type
4. Determine Response Formatand Content
5. Send Response to Client

